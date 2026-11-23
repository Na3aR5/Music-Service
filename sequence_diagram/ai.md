# Sequence Diagram - AI

## Task for AI

Загальні вимоги до задачі: <br>
Зробити sequence-діаграму на КОЖЕН сценарій обраного UC у синтаксисі mermaid sequenceDiagram — актори, компоненти/об'єкти, повідомлення й return-повідомлення, activation bars, комбіновані фрагменти (opt/alt/loop/par). Sequence-діаграма для кожного сценарію, детально; у кожній — щонайменше один фрагмент alt/opt/loop; реалістичні назви дій (submitOrder(), validateUser()).
Activation bar учасника, що працює поза фрагментом або в кількох гілках, починається до і закінчується після умовного фрагмента — не обривається в одній гілці.

Обраний Use Case: <br>
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

Конкретна задача: <br>
Проаналізувати даний Use Case та спираючись на вимоги описані вище, зробити sequence-diagram для кожного сценарію, а саме: Scenario: Playlist creation, Scenario: Playlist deletion,
Scenario: Playlist track addition, Scenario: Playlist track removal, Scenario: Change playlist tracks order, Scenario: Change other user's playlist state denial.

Аудит для перевірки: <br>
- Всі діаграми відповідають вимогам, які описані вище.
- Учасники відповідають сутностям/сервісам ER-моделі
- Кроки збігаються зі сценаріями Gherkin даного Use Case.

ER-diagram для перевірки: <br>
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
