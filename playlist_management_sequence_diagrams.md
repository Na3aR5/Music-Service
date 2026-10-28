# Sequence-діаграми для Use Case: Playlists management

## 1. Прийняті архітектурні припущення

Діаграми побудовані на основі наданого Use Case та ER-моделі. ER-модель визначає сутності `User`, `Playlist`, `PlaylistTrack` і `Track`, але не задає конкретних сервісів, контролерів або репозиторіїв. Тому такі компоненти, як `PlaylistController`, `PlaylistService`, `PlaylistTrackService` та репозиторії, є архітектурними припущеннями для демонстрації взаємодії.

- `User` у діаграмах позначає автентифікованого користувача — актора.
- `Playlist` відповідає сутності плейлиста та полю `Playlist.user_id`.
- `PlaylistTrack` відповідає таблиці зв'язку між плейлистом і треком та містить `position`.
- `Track` відповідає сутності треку, який зберігається в системі незалежно від включення до плейлиста.
- Повідомлення з префіксом `return` позначають відповіді викликаних компонентів. У Mermaid вони подані пунктирними стрілками `-->`.
- Activation bars позначено через `activate` / `deactivate`. Для учасників, що працюють до й після комбінованого фрагмента, активація охоплює весь відповідний фрагмент.
- У кожній діаграмі є щонайменше один комбінований фрагмент `alt`, `opt` або `loop`.

> **Примітка щодо видалення плейлиста.** У сценарії Playlist deletion очікуваний результат сформульовано як видалення зв'язку між плейлистом і користувачем. Оскільки в ER-моделі цей зв'язок представлено полем `Playlist.user_id`, діаграма моделює від'єднання плейлиста від користувача (наприклад, встановлення `user_id` у `NULL`), а не фізичне видалення самого запису `Playlist`. Якщо за правилами застосунку видалення має означати повне видалення плейлиста, цю операцію потрібно змінити відповідно.

---

## 2. Scenario: Playlist creation

**Умова:** автентифікований користувач створює плейлист із непорожньою назвою. Система створює запис плейлиста та пов'язує його з обліковим записом користувача.

```mermaid
sequenceDiagram
    actor User
    participant PC as PlaylistController
    participant PS as PlaylistService
    participant PR as PlaylistRepository
    participant DB as Database

    User->>PC: createPlaylist(name, description)
    activate PC
    PC->>PS: createPlaylist(userId, name, description)
    activate PS
    PS->>PS: validatePlaylistName(name)

    alt Назва непорожня
        PS-->>PS: return validationSuccess
        PS->>PR: savePlaylist(userId, name, description)
        activate PR
        PR->>DB: INSERT Playlist(user_id, name, description)
        activate DB
        DB-->>PR: return playlistId
        deactivate DB
        PR-->>PS: return createdPlaylist
        deactivate PR
        PS-->>PC: return createdPlaylist
        PC-->>User: showPlaylistCreated(playlistId)
    else Назва порожня
        PS-->>PS: return validationError
        PS-->>PC: return error("Playlist name is required")
        PC-->>User: showValidationError()
    end

    deactivate PS
    deactivate PC
```

---

## 3. Scenario: Playlist deletion

**Умова:** користувач має створений плейлист і видаляє його. Відповідно до буквального формулювання сценарію система видаляє зв'язок між плейлистом і користувачем.

```mermaid
sequenceDiagram
    actor User
    participant PC as PlaylistController
    participant PS as PlaylistService
    participant PR as PlaylistRepository
    participant DB as Database

    User->>PC: deletePlaylist(playlistId)
    activate PC
    PC->>PS: detachPlaylistFromUser(userId, playlistId)
    activate PS

    PS->>PR: findById(playlistId)
    activate PR
    PR->>DB: SELECT Playlist WHERE playlist_id = playlistId
    activate DB
    DB-->>PR: return playlist
    deactivate DB
    PR-->>PS: return playlist
    deactivate PR

    alt Плейлист існує та належить користувачу
        PS->>PR: detachOwner(playlistId, userId)
        activate PR
        PR->>DB: UPDATE Playlist SET user_id = NULL WHERE playlist_id = playlistId AND user_id = userId
        activate DB
        DB-->>PR: return rowsUpdated
        deactivate DB
        PR-->>PS: return detachResult
        deactivate PR
        PS-->>PC: return deletionSuccess
        PC-->>User: showPlaylistDeleted()
    else Плейлист не знайдено або користувач не є власником
        PS-->>PC: return deletionDenied
        PC-->>User: showDeletionError()
    end

    deactivate PS
    deactivate PC
```

---

## 4. Scenario: Playlist track addition

**Умова:** користувач додає до свого плейлиста трек, який уже існує в системі. Система створює зв'язок `PlaylistTrack` і зберігає позицію треку.

```mermaid
sequenceDiagram
    actor User
    participant PC as PlaylistController
    participant PTS as PlaylistTrackService
    participant PR as PlaylistRepository
    participant TR as TrackRepository
    participant PTR as PlaylistTrackRepository
    participant DB as Database

    User->>PC: addTrackToPlaylist(playlistId, trackId)
    activate PC
    PC->>PTS: addTrack(userId, playlistId, trackId)
    activate PTS

    PTS->>PR: findOwnedPlaylist(playlistId, userId)
    activate PR
    PR->>DB: SELECT Playlist WHERE playlist_id = playlistId AND user_id = userId
    activate DB
    DB-->>PR: return playlist
    deactivate DB
    PR-->>PTS: return playlist
    deactivate PR

    PTS->>TR: findById(trackId)
    activate TR
    TR->>DB: SELECT Track WHERE track_id = trackId
    activate DB
    DB-->>TR: return track
    deactivate DB
    TR-->>PTS: return track
    deactivate TR

    alt Плейлист належить користувачу і трек існує
        PTS->>PTR: getNextPosition(playlistId)
        activate PTR
        PTR->>DB: SELECT MAX(position) FROM PlaylistTrack WHERE playlist_id = playlistId
        activate DB
        DB-->>PTR: return maxPosition
        deactivate DB
        PTR-->>PTS: return nextPosition
        deactivate PTR

        PTS->>PTR: addPlaylistTrack(playlistId, trackId, nextPosition)
        activate PTR
        PTR->>DB: INSERT PlaylistTrack(playlist_id, track_id, position)
        activate DB
        DB-->>PTR: return playlistTrackId
        deactivate DB
        PTR-->>PTS: return createdPlaylistTrack
        deactivate PTR
        PTS-->>PC: return additionSuccess
        PC-->>User: showTrackAdded(trackId, nextPosition)
    else Плейлист недоступний або трек не існує
        PTS-->>PC: return additionFailed
        PC-->>User: showAdditionError()
    end

    deactivate PTS
    deactivate PC
```

---

## 5. Scenario: Playlist track removal

**Умова:** плейлист містить вибраний трек. Система видаляє лише зв'язок між плейлистом і треком. Сам запис `Track` залишається в системі.

```mermaid
sequenceDiagram
    actor User
    participant PC as PlaylistController
    participant PTS as PlaylistTrackService
    participant PR as PlaylistRepository
    participant PTR as PlaylistTrackRepository
    participant DB as Database

    User->>PC: removeTrackFromPlaylist(playlistId, trackId)
    activate PC
    PC->>PTS: removeTrack(userId, playlistId, trackId)
    activate PTS

    PTS->>PR: findOwnedPlaylist(playlistId, userId)
    activate PR
    PR->>DB: SELECT Playlist WHERE playlist_id = playlistId AND user_id = userId
    activate DB
    DB-->>PR: return playlist
    deactivate DB
    PR-->>PTS: return playlist
    deactivate PR

    PTS->>PTR: findPlaylistTrack(playlistId, trackId)
    activate PTR
    PTR->>DB: SELECT PlaylistTrack WHERE playlist_id = playlistId AND track_id = trackId
    activate DB
    DB-->>PTR: return playlistTrack
    deactivate DB
    PTR-->>PTS: return playlistTrack
    deactivate PTR

    alt Плейлист належить користувачу та містить вибраний трек
        PTS->>PTR: deletePlaylistTrack(playlistId, trackId)
        activate PTR
        PTR->>DB: DELETE FROM PlaylistTrack WHERE playlist_id = playlistId AND track_id = trackId
        activate DB
        DB-->>PTR: return rowsDeleted
        deactivate DB
        PTR-->>PTS: return removalSuccess
        deactivate PTR
        Note over PTS,DB: Запис Track не видаляється
        PTS-->>PC: return removalSuccess
        PC-->>User: showTrackRemoved(trackId)
    else Плейлист недоступний або трек у ньому відсутній
        PTS-->>PC: return removalFailed
        PC-->>User: showRemovalError()
    end

    deactivate PTS
    deactivate PC
```

---

## 6. Scenario: Change playlist tracks order

**Умова:** плейлист містить щонайменше два треки. Користувач змінює порядок, а система зберігає нові значення `PlaylistTrack.position`. Усі треки залишаються в плейлисті.

```mermaid
sequenceDiagram
    actor User
    participant PC as PlaylistController
    participant PTS as PlaylistTrackService
    participant PR as PlaylistRepository
    participant PTR as PlaylistTrackRepository
    participant DB as Database

    User->>PC: changeTrackOrder(playlistId, orderedTrackIds)
    activate PC
    PC->>PTS: updateTrackPositions(userId, playlistId, orderedTrackIds)
    activate PTS

    PTS->>PR: findOwnedPlaylist(playlistId, userId)
    activate PR
    PR->>DB: SELECT Playlist WHERE playlist_id = playlistId AND user_id = userId
    activate DB
    DB-->>PR: return playlist
    deactivate DB
    PR-->>PTS: return playlist
    deactivate PR

    PTS->>PTR: findTracksInPlaylist(playlistId)
    activate PTR
    PTR->>DB: SELECT track_id, position FROM PlaylistTrack WHERE playlist_id = playlistId ORDER BY position
    activate DB
    DB-->>PTR: return currentTracks
    deactivate DB
    PTR-->>PTS: return currentTracks
    deactivate PTR

    alt Плейлист належить користувачу, містить щонайменше 2 треки, а новий порядок коректний
        PTS->>PTS: validateTrackOrder(currentTracks, orderedTrackIds)
        loop Для кожного треку в новому порядку
            PTS->>PTR: updatePosition(playlistId, trackId, newPosition)
            activate PTR
            PTR->>DB: UPDATE PlaylistTrack SET position = newPosition WHERE playlist_id = playlistId AND track_id = trackId
            activate DB
            DB-->>PTR: return updateResult
            deactivate DB
            PTR-->>PTS: return positionUpdated
            deactivate PTR
        end
        PTS-->>PC: return orderUpdateSuccess
        PC-->>User: showUpdatedTrackOrder(orderedTrackIds)
    else Плейлист недоступний, має менше 2 треків або порядок некоректний
        PTS-->>PC: return orderUpdateFailed
        PC-->>User: showOrderUpdateError()
    end

    deactivate PTS
    deactivate PC
```

---

## 7. Scenario: Change other user's playlist state denial

**Умова:** автентифікований користувач намагається змінити плейлист іншого користувача. Система відмовляє в операції, а стан плейлиста не змінюється.

```mermaid
sequenceDiagram
    actor User
    participant PC as PlaylistController
    participant PS as PlaylistService
    participant PR as PlaylistRepository
    participant DB as Database

    User->>PC: changePlaylistState(playlistId, operation, payload)
    activate PC
    PC->>PS: authorizePlaylistChange(userId, playlistId, operation, payload)
    activate PS

    PS->>PR: findById(playlistId)
    activate PR
    PR->>DB: SELECT Playlist WHERE playlist_id = playlistId
    activate DB
    DB-->>PR: return playlist
    deactivate DB
    PR-->>PS: return playlist
    deactivate PR

    alt Плейлист належить іншому користувачу
        PS->>PS: verifyOwnership(userId, playlist.user_id)
        PS-->>PC: return accessDenied("Not playlist owner")
        PC-->>User: showAccessDenied()
        Note over PS,DB: Операція зміни не виконується; Playlist і PlaylistTrack залишаються незмінними
    else Плейлист належить поточному користувачу
        PS->>PS: validateRequestedOperation(operation, payload)
        PS-->>PC: return authorizedForChange
        PC-->>User: continueWithAuthorizedOperation()
    end

    deactivate PS
    deactivate PC
```

---

## 8. Перевірка відповідності вимогам

| Вимога | Реалізація |
|---|---|
| Окрема діаграма для кожного сценарію | Створено 6 діаграм |
| Синтаксис Mermaid `sequenceDiagram` | Використано в усіх діаграмах |
| Актори та учасники | Актор `User`, контролер, сервіси, репозиторії та база даних |
| Реалістичні назви операцій | Наприклад, `createPlaylist()`, `addTrack()`, `updatePosition()` |
| Повідомлення-відповіді | Пунктирні стрілки з результатами `return` |
| Activation bars | Використано `activate` і `deactivate` |
| Комбіновані фрагменти | У кожній діаграмі є `alt`; також використано `opt` і `loop` |
| Узгодженість з ER-моделлю | Операції спираються на `Playlist.user_id`, `PlaylistTrack.playlist_id`, `PlaylistTrack.track_id` і `PlaylistTrack.position` |
| Трек не видаляється з системи під час вилучення з плейлиста | Видаляється лише запис `PlaylistTrack` |
| Відмова у зміні чужого плейлиста | Перевіряється власник; за відмови мутація не виконується |

### Важливі зауваження

1. **Playlist deletion:** ER-модель не визначає, чи потрібно фізично видаляти плейлист, чи лише від'єднувати його від користувача. Поточна діаграма відповідає буквальному формулюванню «видаляє зв'язок між плейлистом і обліковим записом». Якщо `Playlist.user_id` є обов'язковим (`NOT NULL`) або бізнес-правило передбачає повне видалення, реалізацію треба скоригувати.
2. **Playlist creation:** перевірка непорожньої назви виконується до збереження. В гілці помилки операція завершується без створення запису.
3. **Playlist track addition:** `nextPosition` у прикладі означає позицію після останнього треку; у реальній реалізації треба визначити початкову позицію та поведінку для порожнього плейлиста.
4. **Change playlist tracks order:** оновлення позицій бажано виконувати в транзакції, щоб уникнути частково застосованого порядку, якщо один з `UPDATE` завершиться помилкою.
5. **Зміна чужого плейлиста:** сценарій показує відмову на перевірці власника до будь-яких змін. Для операцій додавання, видалення й перестановки треків така перевірка має виконуватися на сервері, а не лише в клієнтському інтерфейсі.
6. **Activation bars:** якщо Mermaid-рендерер суворо перевіряє вкладені активації, можна за потреби винести валідацію в окремий `PlaylistValidationService`; у такому разі `activate` / `deactivate` будуть однозначними для кожного учасника.
