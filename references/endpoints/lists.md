# lists endpoints

Extracted from the WeTrakr docs app on 2026-10-01 (changelog 1.0.7). Each route is `https://api.wetrakr.com` plus the path. Send `wetrakr-api-version: 1`.

Public lists and the signed-in user's lists. Privacy, VIP, and locked lists: `references/social.md`.

Auth `oauth` requires `Authorization: Bearer`. `apiKey` requires `wetrakr-api-key`. `optionalOauth` works with the key; a token adds the caller's state. An empty auth on a direct object means the symbols line is the source.

## Index

- `GET /lists/{id}/comments` — Get list comments
- `POST /lists/{id}/copy` — Copy a list
- `GET /lists/{id}/counters` — Count list items
- `GET /lists/{id}/items/{target}` — Get list items
- `DELETE /lists/{id}/like` — Remove a like
- `POST /lists/{id}/like` — Like a list
- `GET /lists/{id}/likes` — Users who liked it
- `GET /lists/{id}/popularity-history` — Get popularity history
- `GET /lists/{id}` — Get a list
- `POST /lists/from-items` — Create a list from items
- `GET /lists/last` — Latest lists
- `GET /lists/trending` — Trending lists
- `POST /sync/lists/{id}/collaborators/remove` — Remove collaborators
- `POST /sync/lists/{id}/collaborators` — Add collaborators
- `POST /sync/lists/{id}/dice-roll` — Roll the dice
- `DELETE /sync/lists/{id}/follow` — Unfollow a list
- `POST /sync/lists/{id}/follow` — Follow a list
- `GET /sync/lists/{id}/item-ids` — Get item ids
- `POST /sync/lists/{id}/items/{item_id}/order/{direction}` — Nudge an item
- `POST /sync/lists/{id}/items/{item_id}/pin` — Pin or unpin an item
- `POST /sync/lists/{id}/items/{item_id}/rank` — Move an item to a position
- `POST /sync/lists/{id}/items/remove` — Remove items from a list
- `POST /sync/lists/{id}/items/reorder` — Reorder items
- `GET /sync/lists/{id}/items` — Get items of your list
- `POST /sync/lists/{id}/items` — Add items to a list
- `POST /sync/lists/{id}/order/{direction}` — Nudge a list
- `POST /sync/lists/{id}/rank` — Move a list to a position
- `DELETE /sync/lists/{id}` — Delete a list
- `GET /sync/lists/{id}` — Get one of your lists
- `PUT /sync/lists/{id}` — Update a list
- `GET /sync/lists/counters` — Count your lists
- `GET /sync/lists/followed` — Get followed lists
- `GET /sync/lists/item/{target}/{item_id}` — Lists containing an item
- `POST /sync/lists/item/{target}/{item_id}` — Save which lists hold an item
- `GET /sync/lists` — Get your lists
- `POST /sync/lists` — Create a list

## GET /lists/{id}/comments

- Docs id: `lists-comments`
- Auth: `optionalOauth`

Comments written about the list, when the owner allows them.

### Responses

- **200** — Array of comment objects.

## POST /lists/{id}/copy

- Docs id: `lists-copy`
- Auth: `optionalOauth`
- Symbols: `oauth`, `vip`

Create a new list of your own from another list, optionally keeping only the items that match a set of filters and types.

### Body

- `name` (string, optional) — Name of the copy. Defaults to the source name.
- `privacy` (string, optional) — public, friends or private. Default private.

### Responses

- **201** — The new list.
- **426** — The user is not VIP.

## GET /lists/{id}/counters

- Docs id: `lists-counters`
- Auth: `optionalOauth`

How many items the list holds per type, plus its comment count.

### Responses

- **200** — Counters per type.

## GET /lists/{id}/items/{target}

- Docs id: `lists-items`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `privacy`, `paginated`

Items of the list, in the order the owner chose by default. Each item is the media object with its rank and pin state. Accepts the same catalog filters as Search, and with a token the personal ones too. The X-Pagination-* headers also carry the total runtime and how many items you have watched.

### Path params

- `target` (string, required) — movies, shows, seasons, episodes, people or all.
- `id` (integer, required) — WeTrakr list id.

### Query

- `limit` (integer, optional) — Items per page, maximum 100. Default 40.
- `sort_by` (string, optional) — editor_rank (the owner's manual order, the default), added, title, name, rating, avg_rating, release_date, runtime, budget, revenue, votes, popularity, people_watched, people_planning, birth_date, my_rating or my_last_watched. Any other value (watched, random...) falls back to the manual order.
- `sort_dir` (string, optional) — asc or desc. Defaults to asc for editor_rank, desc otherwise.
- `title_search` (string, optional) — Narrow the list to titles matching this text.
- `page` (integer, optional) — Page number, starting at 1.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects. Paging state comes in the X-Pagination-* headers.
- **400** — Unknown target.
- **403** — A personal filter was used without a token.

## DELETE /lists/{id}/like

- Docs id: `lists-unlike`
- Auth: `optionalOauth`

Take the like back.

### Responses

- **200** — Like removed.

## POST /lists/{id}/like

- Docs id: `lists-like`
- Auth: `optionalOauth`

Like the list on behalf of the signed-in user.

### Responses

- **200** — Liked.

## GET /lists/{id}/likes

- Docs id: `lists-likes`
- Auth: `optionalOauth`

Users who liked the list, most recent first.

### Responses

- **200** — Array of likes, each with a compact user and the date.

## GET /lists/{id}/popularity-history

- Docs id: `lists-popularity`
- Auth: `optionalOauth`

Daily popularity score and rank of the list over the last N days.

### Query

- `days` (integer, optional) — Window length in days, 1 to 90 (a larger value gives 90; the response echoes the days applied). Default 30.

### Responses

- **200** — One point per day.
- **400** — The id is not a number.

## GET /lists/{id}

- Docs id: `lists-details`
- Auth: `optionalOauth`

One list with its owner, counters, popularity rank and your permissions on it; extended=preview adds the posters of its first items. Collaborators always see the list; everyone else is subject to its privacy.

### Query

- `extended` (string, optional) — Comma-separated field sets. preview adds preview to each list: up to five of its items (in the list order) with title, ids, poster_path and backdrop_path, enough to draw a list card without reading its items. Empty lists carry none. Other values work as in Extended and append.

### Responses

- **200** — The list object.

## POST /lists/from-items

- Docs id: `lists-from-items`
- Auth: `optionalOauth`
- Symbols: `oauth`, `vip`

Create a list and fill it in one call, for example from a selection the user made in your app.

### Body

- `name` (string, required) — List name.
- `description` (string, optional) — Up to 2000 characters.
- `privacy` (string, optional) — public, friends or private. Default private.
- `allow_comments` (boolean, optional) — Default true.

### Responses

- **201** — The new list.
- **400** — Missing name or no valid items.
- **426** — The user is not VIP.

## GET /lists/last

- Docs id: `lists-last`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

The most recently created public lists.

### Query

- `limit` (integer, optional) — Number of lists, 1 to 20. Default 5.
- `extended` (string, optional) — Comma-separated field sets. preview adds preview to each list: up to five of its items (in the list order) with title, ids, poster_path and backdrop_path, enough to draw a list card without reading its items. Empty lists carry none. Other values work as in Extended and append.

### Responses

- **200** — Array of list objects, newest first.

## GET /lists/trending

- Docs id: `lists-trending`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

Public lists with the most likes right now.

### Query

- `limit` (integer, optional) — Number of lists, 1 to 20. Default 5.
- `extended` (string, optional) — Comma-separated field sets. preview adds preview to each list: up to five of its items (in the list order) with title, ids, poster_path and backdrop_path, enough to draw a list card without reading its items. Empty lists carry none. Other values work as in Extended and append.

### Responses

- **200** — Array of list objects; with extended=preview, each with a poster preview.

## POST /sync/lists/{id}/collaborators/remove

- Docs id: `account-list-collaborators-remove`
- Auth: `oauth`

Revoke collaborator access for the given users.

### Body

- `collaborators` (array, required) — User ids.

### Responses

- **200** — Who was removed.

## POST /sync/lists/{id}/collaborators

- Docs id: `account-list-collaborators-add`
- Auth: `oauth`
- Symbols: `oauth`, `vip`

Invite users to add items to your list. Collaborators always see the list regardless of its privacy.

### Body

- `collaborators` (array, required) — User ids.

### Responses

- **200** — Who was added.
- **426** — The user is not VIP.

## POST /sync/lists/{id}/dice-roll

- Docs id: `account-list-dice`
- Auth: `oauth`
- Symbols: `oauth`, `vip`

Record a dice roll on the list (the "pick something for me" feature). Returns the updated roll counter; picking the item is up to your app. VIP only.

### Responses

- **200** — Roll counted.

## DELETE /sync/lists/{id}/follow

- Docs id: `account-list-unfollow`
- Auth: `oauth`

Stop following the list.

### Responses

- **200** — Unfollowed.

## POST /sync/lists/{id}/follow

- Docs id: `account-list-follow`
- Auth: `oauth`

Follow someone else's list to keep it in your profile. Free for anyone: the plan only matters when you read them back, because [followed lists](#/sync/account-lists-followed) come back locked on a free account.

### Responses

- **201** — Followed.
- **200** — Already following.

## GET /sync/lists/{id}/item-ids

- Docs id: `account-list-item-ids`
- Auth: `oauth`

Just the ids of every item in the list: one row per item with its target, its own WeTrakr id (the season id for a season, the episode id for an episode) and its list_item_id. Season and episode rows also carry show_id. Cheap way to know what is already in it.

### Responses

- **200** — One row per item.

## POST /sync/lists/{id}/items/{item_id}/order/{direction}

- Docs id: `account-list-item-order`
- Auth: `oauth`

Move the item one position up or down.

### Path params

- `item_id` (string, required) — Id of the entry in the list, the list_item_id each item carries in Get list items. Not the id of the movie or show.
- `direction` (string, required) — up or down.
- `id` (integer, required) — WeTrakr list id.

### Responses

- **200** — Moved.
- **400** — Already at the edge.

## POST /sync/lists/{id}/items/{item_id}/pin

- Docs id: `account-list-item-pin`
- Auth: `oauth`

Toggle the pin on an item. Pinned items stay on top regardless of the sort.

### Path params

- `item_id` (string, required) — Id of the entry in the list, the list_item_id each item carries in Get list items. Not the id of the movie or show.
- `id` (integer, required) — WeTrakr list id.

### Body

- `target` (string, required) — movie, show, season, episode or person.

### Responses

- **200** — Toggled.

## POST /sync/lists/{id}/items/{item_id}/rank

- Docs id: `account-list-item-rank`
- Auth: `oauth`

Put the item at the given rank and shift the others.

### Path params

- `item_id` (string, required) — Id of the entry in the list, the list_item_id each item carries in Get list items. Not the id of the movie or show.
- `id` (integer, required) — WeTrakr list id.

### Body

- `rank` (integer, required) — New position, starting at 1.

### Responses

- **200** — Rank updated.
- **400** — Invalid rank.

## POST /sync/lists/{id}/items/remove

- Docs id: `account-list-items-remove`
- Auth: `oauth`

Remove the given items. Ranks of the remaining items are renumbered.

### Responses

- **200** — Counts per type.

## POST /sync/lists/{id}/items/reorder

- Docs id: `account-list-items-reorder`
- Auth: `oauth`

Reorder many items in one call: send their list_item_ids in the new order. Those items take the first positions in that order and the rest keep their order after them. One write, and the journal records a lists updated for the list.

### Path params

- `id` (integer, required) — WeTrakr list id.

### Body

- `items` (array, required) — list_item_ids in the new order, up to 5000. Each at most once.

### Responses

- **200** — Items reordered: how many changed position and the size of the list.
- **400** — The error says which: INVALID_PARAMETER (items is missing or not an array), INVALID_ITEM_ID (a value that is not a list_item_id) or DUPLICATE_ITEM (one appears twice).
- **404** — The list is not yours (List not found for this user), or ITEM_NOT_IN_LIST: some items are not in it, listed under items.
- **413** — TOO_MANY_ITEMS: more than 5,000 items. Each call moves its items to the front, so for a longer order send the chunks last-first: the final chunk first and the first chunk last.

## GET /sync/lists/{id}/items

- Docs id: `account-list-items`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`

Items of one of your lists, all types together. Same parameters and filters as the public items endpoint. A list locked by the plan quota answers 420 with the reason and an upgrade block, not an empty page.

### Query

- `limit` (integer, optional) — Items per page, maximum 100. Default 40.
- `sort_by` (string, optional) — editor_rank (default), added, title, rating, release_date or runtime.
- `sort_dir` (string, optional) — asc or desc.
- `page` (integer, optional) — Page number, starting at 1.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects. Paging state comes in the X-Pagination-* headers.

## POST /sync/lists/{id}/items

- Docs id: `account-list-items-add`
- Auth: `oauth`

Add movies, shows, seasons, episodes or people to the list, many at once. A rank places the item at that position; without it the item goes last. Adding a show adds the show, not its episodes.

### Responses

- **200** — Counts per type plus unresolved items.
- **420** — The list is at its item quota.

## POST /sync/lists/{id}/order/{direction}

- Docs id: `account-list-order`
- Auth: `oauth`

Move the list one position up or down among your lists.

### Path params

- `direction` (string, required) — up or down.
- `id` (integer, required) — WeTrakr list id.

### Responses

- **200** — Moved.

## POST /sync/lists/{id}/rank

- Docs id: `account-list-rank`
- Auth: `oauth`

Set the position of the list among your lists (the manual order).

### Body

- `rank` (integer, required) — New position, starting at 1.

### Responses

- **200** — Rank updated.

## DELETE /sync/lists/{id}

- Docs id: `account-list-delete`
- Auth: `oauth`

Delete the list and its items.

### Responses

- **200** — Deleted.

## GET /sync/lists/{id}

- Docs id: `account-list`
- Auth: `oauth`

One of your lists, or one you collaborate on, with your permissions. Anybody else's list, public or not, answers 404 here: read it with GET /lists/{id}.

### Query

- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — The list object.
- **404** — No such list, or it is not yours and you do not collaborate on it.

## PUT /sync/lists/{id}

- Docs id: `account-list-update`
- Auth: `oauth`

Change any of the list settings. Only the fields you send are updated.

### Responses

- **200** — The updated list.
- **400** — Description too long.

## GET /sync/lists/counters

- Docs id: `account-lists-counters`
- Auth: `oauth`

How many lists you own and how many items they hold, against your plan quota.

### Responses

- **200** — Counters.

## GET /sync/lists/followed

- Docs id: `account-lists-followed`
- Auth: `oauth`
- Symbols: `oauth`, `vip`

The lists the user follows. Reading them is VIP: on a free account the call still answers 200 and still names every list, but each one comes back with locked true and locked_reason followed, and extended=preview returns no items. Following a list is free; seeing what is in it is not.

### Query

- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of list objects. On a VIP account.
- **200** — The same call on a free account: every list flagged, no preview.
- **200** — You follow no lists: an empty array.

## GET /sync/lists/item/{target}/{item_id}

- Docs id: `account-lists-for-item`
- Auth: `oauth`

Your lists (and the ones you collaborate on), each flagged with whether it already contains the item. Built for an "add to list" picker.

### Path params

- `target` (string, required) — movie, show, season, episode or person.
- `item_id` (integer, required) — Id of the item.

### Query

- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Your lists with an included flag.
- **404** — No title of that type with that id.

## POST /sync/lists/item/{target}/{item_id}

- Docs id: `account-lists-for-item-set`
- Auth: `oauth`

The other half of the "add to list" picker: send the whole picker back and the item is added to every list you marked and removed from every list you unmarked, in one call. You do not send what changed, you send the state you want; a list you leave out of the body is not touched.

### Path params

- `target` (string, required) — movie, show, season, episode or person.
- `item_id` (integer, required) — Id of the item.

### Responses

- **200** — What actually changed. added and removed carry the lists that moved, so you can refresh them without asking again.
- **400** — Malformed body.
- **404** — None of the ids is a list you can write to.

## GET /sync/lists

- Docs id: `account-lists`
- Auth: `oauth`
- Symbols: `oauth`, `sync`

Every list you own, in your manual order by default. Lists beyond your plan quota come back with locked: true.

### Query

- `sort` (string, optional) — manual (default), name, items or updated.
- `dir` (string, optional) — asc or desc. Defaults to asc for manual and name, desc otherwise.
- `from_date` (datetime, optional) — Only lists updated after this ISO date.
- `extended` (string, optional) — Comma-separated field sets. preview adds preview to each list: up to five of its items (in the list order) with title, ids, poster_path and backdrop_path, enough to draw a list card without reading its items. Empty lists carry none. Other values work as in Extended and append.

### Responses

- **200** — Array of list objects.

## POST /sync/lists

- Docs id: `account-lists-create`
- Auth: `oauth`

Create an empty list. Free accounts can own 5 lists of up to 250 items. The 420 message quotes a VIP ceiling of 40 lists, but there is no tier above VIP yet and that ceiling is not enforced.

### Responses

- **201** — The new list.
- **400** — Missing name, a name that is not text or longer than 100 characters, or a description too long.
- **420** — Plan quota reached.
