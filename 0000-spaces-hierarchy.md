# Circles Spaces Hierarchy

Circles creates its own Matrix Space to contain its rooms.
This gives us a way to find our rooms, and it prevents us from polluting
the list of rooms in Element and other chat clients if/when a user chooses
to use a single account for both Circles and regular Matrix chat.


| Level | Purpose                       | Room Name        | Room Type                | Parent          | Join Rule |
| ----- | -------                       | ---------        | ---------                | ------          | --------- |
|     1 | Top-level space               | "Circles"        | m.space                  | None            | Private   |
|     2 | Container for circle rooms    | "My Circles"     | m.space                  | "Circles"       | Private   |
|     3 | Container for a single circle | e.g. "Friends"   | m.space                  | "My Circles"    | Private   |
|     4 | The user's "wall" room        | e.g. "Friends"   | org.futo.social.timeline | "Friends" space | Invite    |
|     4 | Other user's "wall" room      | e.g. "Friends"   | org.futo.social.timeline | "Friends" space | Invite    |
|     2 | Container for group rooms     | "My Groups"      | m.space                  | "Circles"       | Private   |
|     3 | Group room                    | e.g. "Book club" | org.futo.social.group    | "My Groups"     | Invite    |
