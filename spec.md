# Specification — Music Service

## 1. Intent

The goal is to model the data of a music streaming platform where users can listen to music, create playlists, and artists can publish their own tracks and albums through an artist profile associated with their user account.

The ER model should describe the main entities, their attributes, relationships, cardinalities, primary keys, and foreign keys required to represent these core platform features.

## 2. Entities

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

Represents a playable music track.

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

## 3. Relationships

* A **User** may have zero or one **Artist** profile. An **Artist** profile belongs to exactly one **User**.
* An **Artist** may publish zero or many **Albums**. Each **Album** belongs to exactly one **Artist**.
* An **Artist** may publish zero or many **Tracks**. Each **Track** belongs to exactly one **Artist**.
* An **Album** may contain zero or many **Tracks**. A **Track** may belong to zero or one **Album**.
* A **User** may create zero or many **Playlists**. Each **Playlist** belongs to exactly one **User**.
* A **Playlist** contains zero or many **Tracks**, and a **Track** may be included in zero or many **Playlists**. The `PlaylistTrack` entity represents this relationship because the relationship has its own attributes (`position`, `added_at`).

## 4. Acceptance criteria

1. The model contains the entities **User, Artist, Album, Track, Playlist, PlaylistTrack, and Favourite**.
2. Every entity has a clearly defined **primary key (PK)**.
3. Every foreign key (FK) references an existing entity's primary key.
4. The model represents the relationship between a user and their artist profile with a **0..1 to 1** cardinality.
5. The model represents that an artist can publish multiple albums and tracks.
6. The model represents that an album can contain multiple tracks, while a track belongs to at most one album.
7. The model represents playlists owned by users.
8. The model represents the many-to-many relationship between playlists and tracks through `PlaylistTrack`, which contains relationship-specific attributes.
10. The model contains no physical database implementation such as SQL DDL, ORM classes, indexes, or database-specific types.
11. The resulting ER diagram clearly shows all entities, PKs, FKs, relationships, and their cardinalities.
12. The model is consistent with the domain description in `README.md`.
