# Circles Spaces Hierarchy

Circles creates its own Matrix Space to contain its rooms.
This gives us a way to find our rooms, and it prevents us from polluting
the list of rooms in Element and other chat clients if/when a user chooses
to use a single account for both Circles and regular Matrix chat.


| Level | Purpose                       | Room Name            | Room Type                     | Room Tag               | Parent               | Join Rule      |
| ----- | -------                       | ---------            | ---------                     | --------               | ------               | ---------      |
|     1 | Top-level space               | "Circles"            | m.space                       | org.futo.space.root    | None                 | Private        |
|     2 | Container for circle rooms    | "My Circles"         | m.space                       | org.futo.space.circles | "Circles"            | Private        |
|     3 | Container for a single circle | e.g. "Friends"       | m.space                       | org.futo.social.circle | "My Circles"         | Private        |
|     4 | The user's "wall" room        | e.g. "Friends"       | org.futo.social.timeline      | None                   | "Friends" space      | Invite / Knock |
|     4 | Other user's "wall" room      | e.g. "Friends"       | org.futo.social.timeline      | None                   | "Friends" space      | Invite / Knock |
|     2 | Container for group rooms     | "My Groups"          | m.space                       | org.futo.space.groups  | "Circles"            | Private        |
|     3 | Group room                    | e.g. "Book club"     | org.futo.social.group         | None                   | "My Groups"          | Invite / Knock |
|     2 | Profile space                 | User display name    | org.futo.social.profile_space | None                   | "Circles"            | Invite / Knock |
|     2 | Container for gallery rooms   | "My Photo Galleries" | m.space                       | org.futo.space.photos  | "Circles"            | Private        |
|     3 | Gallery room                  | e.g. "Photos"        | org.futo.social.gallery       | None                   | "My Photo Galleries" | Private        |
