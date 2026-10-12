# Requirements and Use Cases - Specification

## Intent

Define the system's actors, functional requirements using EARS, user stories, use cases, acceptance scenarios in Gherkin, a use-case diagram, and traceability between requirements and scenarios.

## Actors

### Listener
  - Type: Person / User
  - Main goals: discover and play music, manage personal playlists, follow artists.

### Artist
  - Type: Person / Specialized Platform User
  - Main goals: manage an artist profile, publish tracks and albums, inspect monthly statistics.

### Audio Delivery Service
  - Type: External System
  - Main goals: deliver audio data required for playback.

### Monthly Statistics Scheduler
  - Type: External System
  - Main goals: trigger monthly statistics processing.

## EARS Functional Requirements

### Account Access
  - FR-01: WHEN a user submits login credentials, the system SHALL authenticate the user only if the submitted credentials are valid and SHALL reject the authentication attempt otherwise.
  - FR-02: WHILE a user is not authenticated, the system SHALL NOT allow that user to perform protected operations, including modifying playlists, publishing content, following artists, or accessing account-specific management functions.

### Music Playback
  - FR-03: WHEN an authenticated user requests playback of an available track, the system SHALL request the track's audio resource and SHALL initiate playback when the resource is successfully available.
  - FR-04: IF the requested track's audio resource is unavailable or the audio delivery service reports a playback error, THEN the system SHALL inform the user that playback could not be started.

### Playlist Management
  - FR-05: WHEN an authenticated user submits a valid playlist name, the system SHALL create a playlist owned by that user and persist its name and ownership relationship.
  - FR-06: WHEN a playlist owner requests adding an existing track to their playlist, the system SHALL create a playlist-track association with a valid position and SHALL preserve the relationship between the playlist and the track.
  - FR-07: WHEN a playlist owner requests removing a track from their playlist, the system SHALL remove the corresponding playlist-track association without deleting the underlying track.
  - FR-08: IF a user attempts to modify a playlist that they do not own, THEN the system SHALL reject the operation and SHALL NOT modify the playlist.
  - FR-09: WHEN a playlist owner submits a valid new order for tracks in their playlist, the system SHALL persist the updated positions without changing track ownership or deleting the tracks.

### Artist Profiles and Publishing
  - FR-10: WHEN an authenticated user without an artist profile requests artist profile creation and provides valid profile information, the system SHALL create an artist profile associated with that user.
  - FR-11: IF a user who already has an artist profile attempts to create another artist profile, THEN the system SHALL reject the request and SHALL preserve the existing profile.
  - FR-12: WHEN an authenticated user who owns an artist profile submits valid album information, the system SHALL create an album associated with that artist profile.
  - FR-13: WHEN an authenticated user who owns an artist profile submits valid track information and an available audio resource, the system SHALL register a track associated with that artist.
  - FR-14: IF a track is associated with an album, THEN the system SHALL ensure that the track's artist and the album's artist are the same before saving the association.

### Artist Following
  - FR-15: WHEN an authenticated user requests to follow an existing artist they do not already follow, the system SHALL create an ArtistFollower relationship between that user and artist.
  - FR-16: IF an authenticated user attempts to follow the same artist more than once, THEN the system SHALL preserve at most one active follow relationship for that user-artist pair.
  - FR-17: WHEN an authenticated user requests to unfollow an artist they currently follow, the system SHALL remove the corresponding ArtistFollower relationship without deleting the artist or the user's account.

### Monthly Statistics
  - FR-18: WHEN an authorized artist requests monthly statistics for a month for which statistics have been recorded, the system SHALL display the corresponding `ArtistMonthlyStats.play_count` value.
  - FR-19: IF no statistics record exists for the requested artist and month, THEN the system SHALL indicate that statistics are unavailable for that month and SHALL NOT present a fabricated play count.
  - FR-20: WHEN monthly statistics are recorded or updated, the system SHALL ensure that play_count is a non-negative integer representing the total number of plays across all tracks belonging to the corresponding artist during the specified month.
