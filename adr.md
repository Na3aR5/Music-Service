# Architecture Decision Records

## Record 1

### Context

The platform allows users to create an artist profile and publish their own music. Not every user is necessarily an artist.
The model therefore needs to distinguish a regular platform user from an artist while still allowing an artist profile to belong to an existing user account.

### Decision 1

Combine `User` and `Artist` as a one entity when user created account.

Pros:
- User doesn't need to create separate account to become an artits.

Cons:
- All of users are artists. If user doesn't want to be an artist, he cannot delete or remove his artist profile information.
- `Playlist` references `User` and `Album` references `User`. That creates ambigiuty. Is `Playlist` was created by user artist profile or `Album`? 
- If in future, there will be feature, that people can listen to music in a `guest` mode, meaning account creation is unnecessary for this type of activity, that would be a problem, since artists profile must be relative to specific user account.

### Decision 2

Separate `Artist` from `User`

Pros:
- Artist profile is unnecessary to create, if user doesn't want to.
- Not all `User` is an `Artist`, meanwhile each `Artist` is a `User`.
- `Album` and `Track` reference to an `Artist`, meanwhile `Playlist` references `User`. That clearly defines each entity relationships.

Cons:
- User need to create artist account to became artist.

## Decided

Decision 2 was chosen. It clearly defines difference between `Artist` and `User`. A very small part of users actually are artists, making this decision a way more perspective.
All of artists information and activity is relative only to `Artist`.
