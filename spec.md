# Specification — Music Service

## 1. Intent

The goal is to model the data of a music streaming platform where users can listen to music, create and manage their own playlists, follow artists, and artists can publish their own tracks and albums through an artist profile associated with their user account.

The model should also represent monthly listening statistics for artists based on the number of plays across all of their tracks.

The ER model should describe the main entities, their attributes, relationships, cardinalities, primary keys, and foreign keys required to represent these platform features.

## 2. Entities and attributes

### User

Represents a platform user.

* `user_id` — **PK**
* `username`
* `email`
* `password_hash`
* `created_at`

### Artist

Represents an artist profile created by a user.

* `artist_id` — **PK**
* `user_id` — **FK → User.user_id**
* `display_name`
* `bio`

### Album

Represents a collection of tracks published by an artist.

* `album_id` — **PK**
* `artist_id` — **FK → Artist.artist_id**
* `title`
* `release_date`
* `cover_url`

### Track

Represents a playable music track published by an artist.

* `track_id` — **PK**
* `artist_id` — **FK → Artist.artist_id**
* `album_id` — **FK → Album.album_id**
* `title`
* `duration`
* `audio_url`
* `release_date`

### Playlist

Represents a collection of tracks created and managed by a user.

* `playlist_id` — **PK**
* `user_id` — **FK → User.user_id**
* `name`
* `description`
* `created_at`

### PlaylistTrack

Represents a track included in a playlist and stores attributes specific to this relationship.

* `playlist_track_id` — **PK**
* `playlist_id` — **FK → Playlist.playlist_id**
* `track_id` — **FK → Track.track_id**
* `position`
* `added_at`

### ArtistFollower

Represents a user following an artist.

* `user_id` — **PK, FK → User.user_id**
* `artist_id` — **PK, FK → Artist.artist_id**
* `followed_at`

### ArtistMonthlyStats

Represents an artist's listening statistics for a particular month.

* `stats_id` — **PK**
* `artist_id` — **FK → Artist.artist_id**
* `month`
* `play_count`

`play_count` represents the total number of plays across all tracks belonging to the artist during the specified month.

## 3. Relationships

* A **User** may have zero or one **Artist** profile. Each **Artist** profile belongs to exactly one **User**.
* An **Artist** may publish zero or many **Albums**. Each **Album** belongs to exactly one **Artist**.
* An **Artist** may publish zero or many **Tracks**. Each **Track** belongs to exactly one **Artist**.
* An **Album** may contain zero or many **Tracks**. A **Track** may belong to zero or one **Album**.
* A **User** may create zero or many **Playlists**. Each **Playlist** belongs to exactly one **User**.
* A **Playlist** may contain zero or many **Tracks**, and a **Track** may be included in zero or many **Playlists**. The `PlaylistTrack` entity represents this many-to-many relationship because it has relationship-specific attributes.
* A **User** may follow zero or many **Artists**, and an **Artist** may have zero or many **followers**. The `ArtistFollower` entity represents this many-to-many relationship and stores the time when the follow occurred.
* An **Artist** may have zero or many **ArtistMonthlyStats** records. Each statistics record belongs to exactly one **Artist** and represents the artist's total track plays for a specific month.

## 4. Acceptance criteria

1. The model contains the entities **User, Artist, Album, Track, Playlist, PlaylistTrack, ArtistFollower, and ArtistMonthlyStats**.
2. Every entity has a clearly defined **primary key (PK)**.
3. Every foreign key (FK) references an existing entity's primary key.
4. The model represents the relationship between a user and their artist profile with a **0..1 to 1** cardinality.
5. The model represents that an artist can publish multiple albums and tracks.
6. The model represents that an album can contain multiple tracks, while a track belongs to at most one album.
7. The model represents playlists owned by users.
8. The model represents the many-to-many relationship between playlists and tracks through `PlaylistTrack`, which contains relationship-specific attributes.
9. The model represents the many-to-many relationship between users and artists through `ArtistFollower`.
10. The model allows an artist to have multiple followers.
11. The model represents monthly listening statistics for each artist.
12. `ArtistMonthlyStats.play_count` represents the total number of plays across all tracks belonging to the corresponding artist during a specific month.
13. The model does not contain a favourite-track feature.
14. The model contains no physical database implementation such as SQL DDL, ORM classes, indexes, or database-specific types.
15. The resulting ER diagram clearly shows all entities, PKs, FKs, relationships, and their cardinalities.
16. The model is consistent with the domain description in `README.md`.
