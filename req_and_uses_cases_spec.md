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

## User Stories
  - US-01: As a listener, I want to authenticate so that I can access my account and protected features. (FR-01, FR-02)
  - US-02: As a listener, I want to play a track so that I can listen to music. (FR-03, FR-04)
  - US-03: As a listener, I want to create and manage playlists so that I can organize music according to my preferences. (FR-05, FR-06, FR-07, FR-08, FR-09)
  - US-04: As a user, I want to create an artist profile so that I can publish music as an artist. (FR-10, FR-11)
  - US-05: As an artist, I want to publish albums and tracks so that listeners can access my music. (FR-12, FR-13, FR-14)
  - US-06: As a listener, I want to follow and unfollow artists so that I can manage which artists I follow. (FR-15, FR-16, FR-17)
  - US-07: As an artist, I want to view monthly listening statistics so that I can understand my music's listening activity. (FR-18, FR-19, FR-20)

## Use Cases
language: en

@auth <br>
Feature: Authentication <br>
Background: Let's say the user has an account in the system

  - Scenario: Successful authentication <br>
    Given that the user has the correct credentials <br>
    When a user sends their credentials <br>
    The system then checks that they are correct <br>
    And the system gives the user access to protected functions
    
  - Scenario: Denial of authentication with incorrect data <br>
    Given that the user has an account <br>
    But the user enters incorrect credentials <br>
    When the user tries to log in <br>
    The system then rejects the authentication attempt <br>
    And the system reports an error <br>
    And protected features remain unavailable

@playback <br>
Feature: Playback of music tracks <br>
Background: Suppose the user is authenticated in the system

  - Scenario: Successful playback of the track <br>
    Given that the user has selected an existing track <br>
    And the audio resource of the track is available <br>
    When the user starts playback <br>
    Then the system requests an audio resource <br>
    And the system starts playing the track
    
  - Scenario: Playback of the track is impossible due to the unavailability of the audio resource <br>
    Given that the user chose a track <br>
    And the audio resource of the track is not available <br>
    When the user starts playback <br>
    Then the system does not start playback <br>
    And the system reports that the track cannot be played at this time

@playlist <br>
Feature: Playlists management <br>
Background: Suppose the user is authenticated in the system

  - Scenario: Playlist creation <br>
    When user creates a playlist with non-empty name <br>
    Then system creates new playlist <br> 
    And system creates relationship between playlist and user account
    
  - Scenario: Playlist deletion <br>
    Given that user has a created playlist <br>
    When user deletes created playlist <br> 
    Then system deletes relationship between playlist and user account
    
  - Scenario: Playlist track addition <br>
    Given that user has a created playlist <br>
    When user adds track to playlist <br>
    And track is existing in system <br>
    Then system creates a relationship between playlist and track <br>
    And system stores track position in playlist

  - Scenario: Playlist track removal
    Given that user has a playlist <br>
    And playlist contains selected track <br>
    When user removes a track from playlist <br>
    Then system removes relationship between playlist and track <br>
    And track remains in the system

  - Scenario: Change playlist tracks order <br>
    Given that user has a created playlist <br>
    And playlist contains at least 2 tracks <br>
    When user changes tracks order <br> 
    Then system stores new track positions in playlist <br>
    And all tracks remain in playlist 

  - Scenario: Change other user's playlist state denial
    Given that user is authenticated in system <br>
    And playlist belongs to other authenticated user <br>
    When user tries to change (add, remove, change track) playlist <br>
    Then system denies this operation <br>
    And playlist state remains unchanged

@artist_profile <br>
Feature: Artist profile creation <br>
Background: Suppose the user is authenticated in the system

  - Scenario: Successful artist profile creation <br>
    Given that the user does not yet have an artist profile <br>
    And the user provides correct profile data <br>
    When the user sends a request to create an artist profile <br>
    The system then creates an artist profile <br>
    And the system associates the profile with the user account

  - Scenario: Prohibition of creating a second profile of an artist <br>
    Given that the user already has an artist profile <br>
    When a user tries to create another profile <br>
    Then the system rejects the request <br>
    And the existing profile of the artist remains unchanged

@publishing <br>
Feature: Tracks and albums publishing <br>
Background: Assume user has artist profile

  - Scenario: Album publishing <br>
    Given that the user provides correct album data <br>
    When a user publishes an album <br>
    Then the system creates an album <br>
    And the system links the album to the user's artist profile 

  - Scenario: Track publishing <br>
    Given that the user provides correct track data <br>
    And the audio resource of the track is available <br>
    When a user posts a track without being linked to an album <br>
    The system then creates a track <br>
    And the system links the track to the user's artist profile <br>
    And the track remains available without an album

@artist_follow <br>
Feature: Artist followers management <br>
Background: Assume user has artist profile

  - Scenario: Follow artist <br>
    Given that the user is not yet followed to the selected artist <br>
    When a user follows an artist <br>
    Then the system creates a follow relationship between the user and the artist <br>
    And the system saves the time of following

  - Scenario: Unfollow artist <br>
    Given that the user is following selected artist <br>
    When the user unfollows the artist <br>
    Then the system deletes the follow relationship <br>
    And the profile of the artist remains in the system <br>
    And the user account remains in the system

  - Scenario: Preventing re-follow <br>
    Given that the user is following selected artist <br>
    When the user resends the follow request <br>
    Then the system does not create a follow relationship <br>
    And the existing follow relationship remains active

@artist_stats <br>
Feature: View artist monthly statistics <br>
Background: Assume user has artist profile

  - Scenario: View of available statistics <br>
    Given that the statistics for the selected month have been saved for the artist's profile <br>
    When the user requests statistics for that month <br>
    Then the system displays the number of listens from the statistics record <br>
    And the value corresponds to the saved `ArtistMonthlyStats.play_count`
