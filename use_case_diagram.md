
@startuml

left to right direction

actor "User" as U
actor "Anonymous user" as AU
actor "Authenticated user" as EU
actor "Artist" as AR
actor "Audio Delivery Service" as ADS
actor "Monthly Statistics Scheduler" as MSS

AU -|> U
EU -|> U
AR -|> EU

rectangle "Music Service" {

  usecase "Authenticate" as Auth

  usecase "Play track" as Play
  usecase "Handle playback failure" as PlayError

  usecase "Create playlist" as CreatePlaylist
  usecase "Add track to playlist" as AddTrack
  usecase "Remove track from playlist" as RemoveTrack
  usecase "Reorder playlist tracks" as ReorderTracks

  usecase "Follow artist" as Follow
  usecase "Unfollow artist" as Unfollow

  usecase "Create artist profile" as CreateArtist

  usecase "Publish album" as PublishAlbum
  usecase "Publish track" as PublishTrack

  usecase "View monthly statistics" as ViewStats
  usecase "Prepare monthly statistics" as PrepareStats
}

U --> Auth

EU --> Play
EU --> CreatePlaylist
EU --> AddTrack
EU --> RemoveTrack
EU --> ReorderTracks
EU --> Follow
EU --> Unfollow
EU --> CreateArtist
EU --> ViewStats

AR --> PublishAlbum
AR --> PublishTrack

ADS --> Play
Play ..> PlayError : <<extend>>

MSS --> PrepareStats
PrepareStats ..> ViewStats : <<include>>

@enduml
