# Public and Private Profiles in Circles

This document describes how Circles can extend Matrix to provide more
detailed profiles for users.

We propose to have two levels of user profile data: public and private.
Public profile data is world-readable, in the same way that the user's
display name and avatar URL are now.
Private profiles are stored in access-controlled rooms and are available
only to the members of the room.

## Public Profiles
For public profile data, we take the approach in 
[MSC489](https://github.com/matrix-org/matrix-doc/issues/489),
where the user's profile is a key/value store.
This a simple extension to the standard Matrix profile; where it only
permits two keys, `displayname` and `avatar_url`, we permit keys and
values to be any strings with length 255 or less.

## Private Profiles
Our proposed approach for private profiles builds on an earlier
[draft proposal by henri2h](https://github.com/henri2h/matrix-doc/blob/henri2h-profile-as-space/proposals/zzzz-profile-as-space.md),
which in turn extends another Matrix MSC,
[MSC1769: Extensible profiles as rooms](https://github.com/matrix-org/matrix-spec-proposals/pull/1769).

A private profile for a user is stored in a special Matrix room
with the following properties:

* Room type: `org.futo.social.profile_space`
* Join rule: `knock`
* Encryption: Not encrypted

### Advertising "Public" Circles
The profile room is a Matrix Space room.
A user Alice can make some of her circles visible to all of her contacts by
adding the "wall" rooms for those circles as children of her profile space.

For example, Alice might include the "wall" room where she posts updates to
her "Friends" circle.
Then when a new friend Bob connects with her, Bob can see the room in
her space.
Bob can choose to knock on the advertised room, where he will be able
to see Alice's posts.

### Discovering "Private" Profiles
When a user Bob joins a room, he can use the approach from MSC489 to
advertise his private profile to the other members of the room.

In our example above, when Bob joins Alice's profile space, he can send
an `m.room.member` event containing an extra key `profile_space` with
the room id for his profile space as the value.

Alternatively, if the server supports MSC489-style extended public profiles,
then Bob can also add the `profile_space` to his public profile alongside
his display name and avatar url.
Then any other user can find the room id and knock on his profile space.


## Comparison to Other Approaches

### No Public Aliases
Unlike the earlier proposals, we do not use room aliases to provide 
public labels for user profiles.

Both Henri's document and MSC1769 propose to create a special alias for each
profile room, embedding the user id in the alias in special way.
This is problematic for a few reasons.

* First, it requires that a compliant server must check each new alias to see
if it is in the special format, and if so, it must then check to make sure
the new alias matches the current user's user id.
* Second, malicious users on non-compliant servers can create arbitrary aliases
for the profiles of other users on the same server.
* Third, aliases are public, and some users may not want to advertise their
profile to the whole world.
Circles is a privacy-focused app, so this last point is especially important
for us.

### No Peeking
In the previous proposals, clients use "peeking" to look into profile rooms
without joining them.
According to MSC1769, peeking can be especially challenging over federation.

But in Circles, we do not use peeking.
Profile rooms are access controlled, so users must join the room in order
to see the profile information.




