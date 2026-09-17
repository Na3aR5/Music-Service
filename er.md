# ER Diagram

```mermaid
erDiagram
    User {
        int user_id PK
        string username
        string email
        string password_hash
        datetime created_at
    }

    Artist {
        int artist_id PK
        int user_id FK
        string display_name
        string bio
    }

    Album {
        int album_id PK
        int artist_id FK
        string title
        date release_date
        string cover_url
    }

    Track {
        int track_id PK
        int artist_id FK
        int album_id FK
        string title
        int duration
        string audio_url
        date release_date
    }

    Playlist {
        int playlist_id PK
        int user_id FK
        string name
        string description
        datetime created_at
    }

    PlaylistTrack {
        int playlist_track_id PK
        int playlist_id FK
        int track_id FK
        int position
        datetime added_at
    }

    ArtistFollower {
        int user_id PK, FK
        int artist_id PK, FK
        datetime followed_at
    }

    ArtistMonthlyStats {
        int stats_id PK
        int artist_id FK
        date month
        int play_count
    }

    User ||--o| Artist : "has profile"
    Artist ||--o{ Album : "publishes"
    Artist ||--o{ Track : "publishes"
    Album o|--o{ Track : "contains"

    User ||--o{ Playlist : "creates"
    Playlist ||--o{ PlaylistTrack : "contains"
    Track ||--o{ PlaylistTrack : "included in"

    User ||--o{ ArtistFollower : "follows"
    Artist ||--o{ ArtistFollower : "has followers"

    Artist ||--o{ ArtistMonthlyStats : "has statistics"
