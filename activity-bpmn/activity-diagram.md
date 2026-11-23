# BP-01 — Playlist Management Activity Diagram

This diagram models the process-level behavior of playlist management. It uses a Mermaid flowchart with responsibility lanes to represent a UML Activity Diagram. The Listener initiates an operation; the Music Service performs validation, authorization, and persistence.

```mermaid
flowchart TD
    START((Start)) --> AUTH[Listener is authenticated]
    AUTH --> SELECT[Listener selects a playlist operation]

    SELECT --> CREATE_REQ
    SELECT --> DELETE_REQ
    SELECT --> ADD_REQ
    SELECT --> REMOVE_REQ
    SELECT --> REORDER_REQ
    SELECT --> OTHER_REQ

    subgraph L["Listener"]
      direction TB
      CREATE_REQ[Submit create playlist request]
      DELETE_REQ[Submit delete playlist request]
      ADD_REQ[Submit add track request]
      REMOVE_REQ[Submit remove track request]
      REORDER_REQ[Submit reorder request]
      OTHER_REQ[Submit playlist mutation request]
      SHOW_CREATED[Show creation result]
      SHOW_DELETED[Show deletion result]
      SHOW_ADDED[Show addition result]
      SHOW_REMOVED[Show removal result]
      SHOW_REORDERED[Show reorder result]
      SHOW_DENIED[Show access denied]
      SHOW_ERROR[Show validation or operation error]
    end

    subgraph S["Music Service"]
      direction TB

      C_VALIDATE{Playlist name non-empty?}
      C_SAVE[Create Playlist owned by Listener]
      C_DONE([Creation completed])

      D_FIND[Find Playlist]
      D_OWNER{Playlist exists and Listener owns it?}
      D_DELETE[Atomically delete PlaylistTrack associations and Playlist]
      D_DONE([Deletion completed])

      A_OWN{Playlist exists and is owned by Listener?}
      A_TRACK[Find Track]
      A_EXISTS{Track exists?}
      A_POS[Determine next valid position]
      A_INSERT[Create PlaylistTrack association]
      A_DONE([Addition completed])

      R_OWN{Playlist exists and is owned by Listener?}
      R_ASSOC[Find PlaylistTrack association]
      R_EXISTS{Association exists?}
      R_DELETE[Delete PlaylistTrack association only]
      R_DONE([Removal completed])

      O_OWN{Playlist exists and is owned by Listener?}
      O_LOAD[Load current PlaylistTrack membership and positions]
      O_COUNT{At least 2 tracks?}
      O_VALID{New order is an exact permutation of current membership?}
      O_UPDATE[Update positions in one transaction]
      O_LOOP{More tracks to update?}
      O_DONE([Reordering completed])

      X_FIND[Find target Playlist and owner]
      X_OWNER{Listener owns Playlist?}
      X_ALLOW[Continue with the authorized mutation]
      X_DENY([Reject before mutation])

      ERR([Return operation error])
    end

    CREATE_REQ --> C_VALIDATE
    C_VALIDATE -- Yes --> C_SAVE --> C_DONE --> SHOW_CREATED
    C_VALIDATE -- No --> ERR --> SHOW_ERROR

    DELETE_REQ --> D_FIND --> D_OWNER
    D_OWNER -- Yes --> D_DELETE --> D_DONE --> SHOW_DELETED
    D_OWNER -- No: missing or not owner --> ERR --> SHOW_ERROR

    ADD_REQ --> A_OWN
    A_OWN -- Yes --> A_TRACK --> A_EXISTS
    A_EXISTS -- Yes --> A_POS --> A_INSERT --> A_DONE --> SHOW_ADDED
    A_EXISTS -- No --> ERR --> SHOW_ERROR
    A_OWN -- No --> X_DENY --> SHOW_DENIED

    REMOVE_REQ --> R_OWN
    R_OWN -- Yes --> R_ASSOC --> R_EXISTS
    R_EXISTS -- Yes --> R_DELETE --> R_DONE --> SHOW_REMOVED
    R_EXISTS -- No --> ERR --> SHOW_ERROR
    R_OWN -- No --> X_DENY

    REORDER_REQ --> O_OWN
    O_OWN -- Yes --> O_LOAD --> O_COUNT
    O_COUNT -- Yes --> O_VALID
    O_VALID -- Yes --> O_UPDATE --> O_LOOP
    O_LOOP -- Yes --> O_UPDATE
    O_LOOP -- No --> O_DONE --> SHOW_REORDERED
    O_COUNT -- No --> ERR
    O_VALID -- No --> ERR
    O_OWN -- No --> X_DENY

    OTHER_REQ --> X_FIND --> X_OWNER
    X_OWNER -- Yes --> X_ALLOW
    X_OWNER -- No --> X_DENY --> SHOW_DENIED

    SHOW_CREATED --> FINISH((End))
    SHOW_DELETED --> FINISH
    SHOW_ADDED --> FINISH
    SHOW_REMOVED --> FINISH
    SHOW_REORDERED --> FINISH
    SHOW_DENIED --> FINISH
    SHOW_ERROR --> FINISH
    X_ALLOW --> FINISH
```

## Diagram notes

1. **Atomicity:** deletion and reordering are treated as atomic persistence operations. If persistence fails, the operation must not leave a partial result.
2. **Authorization:** every mutation branch checks ownership before changing data. The generic “other mutation” branch makes the denial Use Case explicit; the concrete add, remove, reorder, and delete branches also contain their own ownership gate.
3. **Track preservation:** removing a track from a playlist deletes only the `PlaylistTrack` association. It does not delete the `Track`.
4. **Reorder loop:** the loop updates each track's position sequentially within one transaction. There are no parallel branches or fork/join nodes.
5. **Error paths:** missing playlists, invalid input, missing tracks/associations, invalid order, insufficient track count, and persistence failures terminate through an error result. Authorization failures terminate through an access-denied result.
6. **No-response behavior:** the diagram models logical process outcomes, not transport timeouts. A client must not report success without a service result. An uncertain timeout after a write requires state reconciliation.
