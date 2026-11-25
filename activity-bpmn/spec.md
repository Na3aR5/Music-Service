# BP-01 — Playlist Management

## 1\. Document purpose

This specification describes the end-to-end business process for managing a listener's playlists in the Music Service. It consolidates the playlist-related Use Cases and the six corresponding sequence diagrams:

1. Playlist creation
2. Playlist deletion
3. Playlist track addition
4. Playlist track removal
5. Change playlist tracks order
6. Denial of changes to another user's playlist

The process-level diagram is provided separately in [`diagram.md`](diagram.md).

## 2\. Selected process notation

**Selected notation: UML Activity Diagram.**

An Activity Diagram is appropriate because this process is a collection of operational workflows with:

* ordered actions;
* decisions and alternative outcomes;
* validation and authorization checks;
* a repeated operation when reordering tracks;
* clear start and end points.

A single State Machine would be less suitable: the subject is not the lifecycle of one persistent object, but a set of user-initiated workflows. BPMN could be used for cross-organizational orchestration, but the supplied Use Cases and sequence diagrams describe application behavior rather than a business process spanning multiple organizations.

The diagram uses two responsibility lanes: **Listener** and **Music Service**. Controller, service, repository, and database components are implementation-level participants shown in the sequence diagrams; they are not separate business actors in this activity model.

## 3\. Scope and participants

### 3.1 In scope

* Create a playlist.
* Delete a playlist physically.
* Add an existing track to a playlist.
* Remove a track from a playlist without deleting the track.
* Change the order of tracks in a playlist.
* Reject an attempt to modify a playlist owned by another user.

### 3.2 Participants and responsibilities

|Participant|Responsibility|
|-|-|
|Listener|Initiates playlist operations and receives their outcomes.|
|Music Service|Authenticates/authorizes requests, validates input, reads and updates persistent data, and reports results.|
|Database|Persists `Playlist` and `PlaylistTrack` records. It is a technical component, not a business actor.|

The process assumes that the Listener is authenticated before entering any protected playlist operation. The system must still enforce ownership on the server for every mutation.

## 4\. Preconditions and postconditions

### 4.1 Preconditions

* The Listener is authenticated.
* The Listener submits the identifier of the target playlist for operations on an existing playlist.
* For adding a track, the submitted track identifier refers to a track that is expected to exist; the service verifies this.
* For reordering, the Listener supplies the requested track order; the service validates it against the playlist's current membership.

### 4.2 Postconditions

|Operation|Successful postcondition|Unsuccessful postcondition|
|-|-|-|
|Create|A new `Playlist` exists and is owned by the Listener.|No playlist is created.|
|Delete|The `Playlist` is physically deleted. Its `PlaylistTrack` associations are also removed.|The playlist remains unchanged.|
|Add track|A `PlaylistTrack` association exists with a valid position.|Playlist membership remains unchanged.|
|Remove track|The selected `PlaylistTrack` association is deleted; the `Track` remains.|Playlist membership remains unchanged.|
|Reorder|New positions are persisted; membership is unchanged.|The prior order remains unchanged.|
|Modify another user's playlist|No mutation occurs; access is denied.|The playlist and its track associations remain unchanged.|

**Deletion semantics (decision):** “Delete Playlist” means physical deletion of the `Playlist` entity, not detachment from its owner. The corresponding `PlaylistTrack` records must also be removed, either explicitly as part of the same transaction or by a defined referential cleanup rule. This is consistent with FR-06's requirement to delete the playlist and with the ER relationship in which every Playlist belongs to exactly one User. The former sequence-diagram behavior `UPDATE Playlist SET user\\\_id = NULL` is obsolete and must not be implemented.

## 5\. Detailed process specification

### 5.1 Create playlist

**Trigger:** Listener submits a request to create a playlist.

**Main flow:**

1. Listener submits the playlist name and optional description.
2. Music Service validates the name.
3. If the name is non-empty, Music Service creates a `Playlist` owned by the authenticated Listener.
4. Music Service returns the created playlist identifier/details.
5. Listener is informed that the playlist was created.

**Alternative flow — invalid name:**

1. If the name is empty or otherwise invalid under the validation rule, the service does not persist a playlist.
2. The service returns a validation error.
3. Listener is informed that a playlist name is required.

**Relevant requirement/use case:** FR-05; Playlist creation scenario.

### 5.2 Delete playlist

**Trigger:** Listener requests deletion of one of their playlists.

**Main flow:**

1. Listener submits `playlistId`.
2. Music Service retrieves the playlist.
3. Music Service verifies that the playlist exists and is owned by the authenticated Listener.
4. Music Service physically deletes the playlist and all of its `PlaylistTrack` associations as one atomic operation.
5. Music Service confirms deletion.
6. Listener is informed that the playlist was deleted.

**Alternative flows:**

* If the playlist does not exist, the service returns a not-found result; no data is changed.
* If the playlist belongs to another user, the service denies the request; no data is changed.
* If persistence fails, the delete operation is rolled back or otherwise left in a well-defined unchanged state, and the service reports failure.

**Relevant requirement/use case:** FR-06 and FR-09; Playlist deletion and denial scenarios.

**Important correction to the source sequence diagram:** replace `detachPlaylistFromUser()` / `detachOwner()` and `UPDATE Playlist SET user\\\_id = NULL` with physical deletion. A suitable logical operation is `deleteOwnedPlaylist(userId, playlistId)`, followed by deletion of the playlist's `PlaylistTrack` rows and then the `Playlist` row, in one transaction. The order may be reversed if the database's referential cleanup behavior is explicitly configured, but the observable result must be the same.

### 5.3 Add track to playlist

**Trigger:** Listener requests adding an existing track to one of their playlists.

**Main flow:**

1. Listener submits `playlistId` and `trackId`.
2. Music Service verifies that the playlist exists and belongs to the Listener.
3. Music Service verifies that the track exists.
4. Music Service determines the next valid position in the playlist.
5. Music Service creates a `PlaylistTrack` association with that position.
6. Listener is informed that the track was added.

**Alternative flows:**

* If the playlist is missing or not owned by the Listener, the service denies the operation.
* If the track does not exist, the service rejects the operation.
* If persistence fails, no successful addition is reported.

**Relevant requirement/use case:** FR-07 and FR-09; Playlist track addition and denial scenarios.

**Position rule to define in implementation:** the specification's sequence diagram uses `MAX(position)` to calculate the next position. The implementation must define the initial position for an empty playlist and ensure that concurrent additions cannot allocate the same position.

### 5.4 Remove track from playlist

**Trigger:** Listener requests removing a track from one of their playlists.

**Main flow:**

1. Listener submits `playlistId` and `trackId`.
2. Music Service verifies playlist ownership.
3. Music Service verifies that the corresponding `PlaylistTrack` association exists.
4. Music Service deletes that association.
5. The underlying `Track` is not deleted.
6. Listener is informed that the track was removed.

**Alternative flows:**

* If the playlist is missing or not owned by the Listener, the service denies the operation.
* If the association does not exist, the service reports that removal could not be completed.
* If persistence fails, no successful removal is reported.

**Relevant requirement/use case:** FR-08 and FR-09; Playlist track removal and denial scenarios.

### 5.5 Change track order

**Trigger:** Listener submits a new order for the tracks in one of their playlists.

**Main flow:**

1. Listener submits `playlistId` and an ordered list of track identifiers.
2. Music Service verifies playlist ownership.
3. Music Service retrieves the playlist's current track membership and positions.
4. Music Service verifies that the playlist contains at least two tracks.
5. Music Service validates that the submitted order is a valid permutation of the current playlist membership: every current track appears exactly once, and no extra track is included.
6. Music Service updates the `position` of each `PlaylistTrack` association.
7. All updates are committed atomically.
8. Listener is informed that the order was updated.

**Alternative flows:**

* If the playlist is missing or not owned by the Listener, the service denies the operation.
* If the playlist has fewer than two tracks, the operation is rejected.
* If the submitted order is invalid, the operation is rejected.
* If any position update fails, the transaction is rolled back so that a partially updated order is not exposed.

**Relevant requirement/use case:** FR-10 and FR-09; Change playlist tracks order and denial scenarios.

**Model refinement:** the exact-permutation validation and transactional update are recommended precision requirements. They make “valid new order” and “persist the updated positions” unambiguous; they should be reflected in the implementation contract.

### 5.6 Deny modification of another user's playlist

**Trigger:** An authenticated Listener attempts to modify a playlist they do not own.

**Main flow:**

1. Listener submits a playlist mutation request.
2. Music Service retrieves the playlist and its owner.
3. Music Service compares the owner identifier with the authenticated user's identifier.
4. If they differ, the service rejects the operation before any mutation.
5. Listener receives an access-denied result.
6. The `Playlist` and its `PlaylistTrack` associations remain unchanged.

**Note:** ownership checks must be performed server-side. Hiding controls in the client interface is not an authorization mechanism.

**Relevant requirement/use case:** FR-09; Change other user's playlist state denial scenario.

## 6\. Message and response contract

The following table identifies the messages exchanged at the process boundary. Internal calls between Controller, Service, Repository, and Database are implementation details and are expanded in the sequence diagrams.

|Message|Sender → Receiver|Purpose|Expected response / no-response behavior|
|-|-|-|-|
|Create playlist request|Listener → Music Service|Submit name and description.|Service returns created playlist or validation/operation error. If no response arrives, client must not display success; it may show a retryable/unknown outcome.|
|Delete playlist request|Listener → Music Service|Request physical deletion by playlist ID.|Service returns deletion confirmation or not-found/forbidden/error. If outcome is uncertain after timeout, client must reconcile by querying playlist existence before retrying.|
|Add track request|Listener → Music Service|Submit playlist ID and track ID.|Service returns addition confirmation or ownership/track/operation error. No success is shown without confirmation.|
|Remove track request|Listener → Music Service|Submit playlist ID and track ID.|Service returns removal confirmation or ownership/association/operation error. No success is shown without confirmation.|
|Reorder request|Listener → Music Service|Submit playlist ID and ordered track IDs.|Service returns order-update confirmation or ownership/validation/operation error. On timeout, service/client must avoid assuming partial success; transaction and reconciliation are required.|
|Modify playlist request|Listener → Music Service|Submit a mutation targeting a playlist.|Service returns authorization result. If denied, no mutation is executed. If no response arrives, client must treat the result as unknown, not as authorization.|
|Operation result|Music Service → Listener|Communicate success or failure.|This is a notification/result, not a request; no acknowledgment is required by the supplied Use Cases.|

### 6.1 No-response policy

The supplied Use Cases and sequence diagrams do not define transport-level timeout behavior. The following is a proposed system-level policy:

* Every request must have a bounded timeout.
* The client must never show success merely because it sent a request.
* Read-only failures can be reported as retryable.
* For a write request that times out after it may have reached the database, the outcome is **unknown**, not necessarily failed. The client/service should reconcile the current state before blindly retrying.
* Reorder updates and playlist deletion must be atomic.
* If a service response is lost after a committed operation, a repeated request should be safe or the client should query the resulting state. Idempotency keys are a possible implementation mechanism, but are not specified in the source requirements.

## 7\. Traceability: Activity Diagram ↔ Use Cases ↔ Sequence Diagrams

|Activity flow|Use Case scenario|Main sequence-diagram operations|
|-|-|-|
|Create playlist|Playlist creation|`createPlaylist()` → `validatePlaylistName()` → `savePlaylist()`|
|Delete playlist|Playlist deletion|`deletePlaylist()` → find/verify owner → delete playlist and its associations|
|Add track|Playlist track addition|`addTrackToPlaylist()` → `findOwnedPlaylist()` → `findById(trackId)` → `getNextPosition()` → `addPlaylistTrack()`|
|Remove track|Playlist track removal|`removeTrackFromPlaylist()` → `findOwnedPlaylist()` → `findPlaylistTrack()` → `deletePlaylistTrack()`|
|Reorder tracks|Change playlist tracks order|`changeTrackOrder()` → `findOwnedPlaylist()` → `findTracksInPlaylist()` → `validateTrackOrder()` → repeated `updatePosition()`|
|Reject foreign playlist mutation|Change other user's playlist state denial|`changePlaylistState()` → `authorizePlaylistChange()` → `findById()` → `verifyOwnership()`|

The sequence-diagram source currently models playlist deletion as detachment. That sequence must be revised as described in §5.2 before the artifacts can be considered fully consistent.

## 8\. Audit

### 8.1 Use Case coverage

|Check|Result|
|-|-|
|Playlist creation represented|Pass|
|Playlist deletion represented as physical deletion|Pass in this specification; source sequence diagram requires correction|
|Add track represented|Pass|
|Remove track represented without deleting Track|Pass|
|Reorder represented|Pass|
|Denial for another user's playlist represented|Pass|
|Success and failure outcomes represented|Pass|

### 8.2 Transition reachability

* Every operation starts from the process entry and the Listener's selected action.
* Each operation has a reachable success path and, where applicable, a reachable validation, not-found, authorization, or persistence-failure path.
* Every branch reaches a result/termination path; there are no dead-end activities.
* The reorder loop is reachable only after ownership, minimum-membership, and order-validation checks succeed.

### 8.3 Conditions

|Condition|Applied to|Expected result|
|-|-|-|
|Playlist name is non-empty|Create|Create only when true; otherwise validation error.|
|Playlist exists|Existing-playlist operations|Continue only when found.|
|Authenticated user owns playlist|All playlist mutations|Continue only when true; otherwise deny without mutation.|
|Track exists|Add|Create association only when found.|
|PlaylistTrack association exists|Remove|Delete association only when found.|
|Playlist has at least two tracks|Reorder|Continue only when true.|
|New order is a permutation of current membership|Reorder|Update positions only when valid.|
|All writes succeed|Delete/reorder and other mutations|Commit success; otherwise roll back or report a well-defined failure.|

### 8.4 Parallel branches and joins

There are **no parallel branches and no fork/join nodes** in this process. Each operation is sequential. Reordering uses a loop over the ordered track list; the loop is not parallel execution. A parallel join would add no value to the supplied workflows and could complicate consistency of position updates.

### 8.5 Message consistency and response audit

* Each of the six Listener-initiated operations in the Activity Diagram maps to the corresponding sequence-diagram request.
* The sequence diagrams expand the Music Service work into Controller/Service/Repository/Database calls.
* Each synchronous internal request has a corresponding return/result in the sequence diagrams.
* The final user-facing success/error notification is one-way; the supplied model does not require a user acknowledgment.
* A missing response must not be interpreted as success. For writes, timeout may leave the outcome unknown and requires reconciliation.
* **Deletion discrepancy:** the existing sequence diagram's `detachPlaylistFromUser()` and `UPDATE user\\\_id = NULL` do not correspond to the decided physical-delete behavior. Replace them with an atomic delete of `PlaylistTrack` associations and `Playlist`, and rename the service operation accordingly.

### 8.6 Design risks and required clarifications

1. **Delete semantics:** resolved here as physical deletion. Update the sequence diagram and implementation accordingly.
2. **Referential cleanup:** define whether deleting a Playlist explicitly deletes `PlaylistTrack` rows or uses a database-level cascade. Either way, no orphaned associations may remain.
3. **Reorder atomicity:** use a transaction so a failure cannot leave only some positions updated.
4. **Concurrent additions:** `MAX(position)` can race when two requests add tracks concurrently. Use an appropriate serialization/locking or conflict-handling strategy.
5. **Position convention:** define whether positions start at zero or one, and how gaps are handled.
6. **Duplicate track membership:** the supplied requirements do not say whether the same Track may appear more than once in one Playlist. Do not impose a restriction without a business decision.
7. **Failure/timeout behavior:** specify API error responses and reconciliation for uncertain write outcomes.
8. **Source-document inconsistencies outside this process:** US-04 references FR-10 for artist-profile creation, although FR-10 concerns playlist ordering; US-07 references FR-19 and FR-20, which are not defined in the supplied FR list. These are outside BP-01 but should be corrected in the requirements document.

## 9\. Acceptance checklist for the final model

* \[x] One process notation selected and justified.
* \[x] All six playlist Use Case scenarios covered.
* \[x] Roles/responsibilities identified.
* \[x] Success and alternative paths specified.
* \[x] Ownership checks precede every playlist mutation.
* \[x] Physical playlist deletion explicitly specified.
* \[x] Track removal preserves the Track entity.
* \[x] Reordering preserves membership and is atomic.
* \[x] No-response behavior documented as a proposed policy.
* \[x] No unreachable branches or unnecessary parallel joins.
* \[x] Activity actions traced to sequence-diagram operations.

