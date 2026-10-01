# Lists, comments, users, account

Parameter tables: `references/endpoints/lists.md`, `references/endpoints/comments.md`, `references/endpoints/account.md`.

Quotas, locked lists, and VIP: `references/conventions.md`.

## Two list APIs

`/lists/...` is the public catalog of lists. `/sync/lists/...` is the signed-in user's lists and the lists they collaborate on or follow. Both are needed. They are not aliases.

| Need | Route |
|---|---|
| Trending or newest public lists | `GET /lists/trending`, `GET /lists/last` |
| Any list, including someone else's public list | `GET /lists/{id}` |
| Items in public order, with catalog filters | `GET /lists/{id}/items/{target}` |
| Counts, likers, like, unlike, comments, popularity history | `/lists/{id}/...` |
| Copy a public list, or build one from a set of items | `POST /lists/{id}/copy`, `POST /lists/from-items` — **VIP** (`426`) |
| Your lists, create, update, delete, reorder, pin, collaborators | `/sync/lists/...` — oauth |
| Which of your lists contain an item, or set that | `GET` and `POST /sync/lists/item/{target}/{item_id}` |
| Lists you follow | `GET /sync/lists/followed` |

`GET /sync/lists/{id}` returns 404 for a list you do not own and do not collaborate on, even if it is public. Read that list with `GET /lists/{id}`.

`extended=preview` works on `GET /lists/{id}` and `GET /sync/lists/{id}`: posters of the first items. Also on Discover trending, popular, and last when `filter_type=list` (up to five items with title, ids, poster, backdrop).

`GET /sync/lists/{id}/item-ids` is the cheap membership check. One row per item: `target`, the item's own WeTrakr id (season id for a season, episode id for an episode, **not** the show id), `list_item_id`, and `show_id` on season and episode rows.

Adding a show adds the show, not its episodes. `rank` inserts at that position. Without `rank` the item goes last. Remove renumbers. Reorder sends `list_item_ids` in the new order. Those items take the first positions. The rest keep their relative order. One write, one journal `lists` updated row. Reorder errors: `INVALID_PARAMETER`, `INVALID_ITEM_ID`, `DUPLICATE_ITEM`, `ITEM_NOT_IN_LIST`, `TOO_MANY_ITEMS`.

Pinned items stay on top of sort. `POST .../pin` toggles. `order/up` and `order/down` nudge one step, for items and for the list among your lists.

Collaborators can add items and can see the list regardless of privacy. Follow is free. **Reading** followed lists is VIP: `GET /sync/lists/followed` is still `200` and still names every list, but each one has `locked: true` and `locked_reason: followed`. `extended=preview` returns no items. Dice roll (`POST /sync/lists/{id}/dice-roll`) is VIP.

Sort on your lists: `manual` (default), `updated`, `created`, `name`, `items`.

Privacy on a list you own hides it from non-owners. A friend with access is the exception called out on `GET /users/{id}/lists`.

## Comments

Public comment routes are under `/comments`. The caller's own comments are under `/sync/comments`.

| Path | Notes |
|---|---|
| `GET /comments/trending/{target}` | Most liked recent comments for that target type. apiKey. |
| `GET /comments/last/{target}` | Most recent. apiKey. |
| `GET /comments/{id}` | One comment. |
| `GET /comments/{id}/replies` | Replies. |
| `POST /comments/{id}/replies` | Reply. oauth. |
| `PATCH /comments/{id}` | Edit text or spoiler flag. Comments removed by moderation cannot be edited. |
| `POST` / `DELETE /comments/{id}/like` | Like. |
| `GET /comments/{id}/likes` | Likers, newest first. |
| `GET /comments/{id}/translate` | Cached translation. If the original is already in that language, it comes back unchanged. |
| `POST` / `DELETE /comments/{id}/hide` | Hide is per user. Hidden comments still appear in lists with `hidden: true` so the UI can collapse them. |

Write with `POST /sync/comments`. Targets: movie, show, season, episode, person, list. Language is detected. At 200 words or more, `is_long` is true and the comment is shown as a review. Text is required unless the comment has a reaction or an attachment. Empty or blank text, and text over 10,000 characters, is `400` `INVALID_PARAMETER` (1.0.7).

`GET /sync/comments/{target}` is the caller's comments, newest first, 20 per page. Replies are omitted unless `include_replies=true`. `is_long=true` or `false` filters reviews vs short comments. The same filter exists on `GET /users/{id}/comments/{target}`.

`POST /sync/comments/remove` deletes the caller's comments on the given items (same batched id body as ratings). `DELETE /sync/comments/{comment_id}` deletes one comment or reply. Deleting a comment deletes its replies.

Counters put replies in `replies`, not in `all`. Every comment list sends `X-Pagination-Item-Count`, including list comments, replies, trending, and last.

List comments (`GET /lists/{id}/comments`) exist only when the owner allows comments.

## Users

`GET /users/{id}` is the public profile: identity, images, social links, location, plan, activity, and content counters. It does **not** include account settings or display preferences. When the profile is unavailable the endpoint page describes the status. Do not treat a profile as the settings object.

Everything else is `GET /users/{id}/...` and follows the same shapes as the caller's `/sync` lists, with privacy:

| Path | Notes |
|---|---|
| `/tracking/{status}/{target}` and `/counters` | `403` `PRIVATE` or `FRIENDS_ONLY` when tracking is hidden. |
| `/ratings/{target}` and `/counters` | Their score is `user_rating`. With your token, `interactions.user.rating` is still **your** score, not theirs. |
| `/shows/{show_id}/episode-ratings` | Map of episode id to the rating they gave. For a season heatmap. |
| `/favorites/{target}` and `/counters` | |
| `/lists` | Public lists, unless you are the owner or a friend with access. Counts on each list. `extended=preview` adds posters. |
| `/likes/{target}` | Lists and comments they liked. |
| `/comments/{target}` and `/counters` | |
| `/following`, `/followers` | `page` and `limit` with `X-Pagination-*`. Omit them and the full list comes back, as before 1.0.7. Following entries include the approval date. |

`POST /users/{id}/follow` follows immediately on a public account. A private account creates a request and the entry stays pending. `DELETE /users/{id}/follow` unfollows or withdraws the request.

`POST /users/{id}/block` removes any follow in both directions, stops them following you, and hides their comments from you. `DELETE /users/{id}/block` lifts it.

Hidden sections used to return `200` with an empty list or zero counters. They now return `403`. An empty list means the section is visible and empty.

## Account

`GET /account/settings` is a fixed field set: who they are, where they are, what they can see, and the preferences that change how dates and titles should be read. A new setting in the WeTrakr apps does not appear here on its own. `POST /account/settings` updates that set. Read the body fields in `references/endpoints/account.md` before sending a key you have not seen there.

| Path | Notes |
|---|---|
| `GET /account/plan-usage` | `used` is real. `limit` is the advertised number. On VIP, `used` may exceed `limit`. |
| `GET /account/last` | Recent activity. |
| `GET /account/stats/week` and `/stats/month` | Watched and rated in that window. |
| `GET /account/stats/all` | VIP. A free account gets `200` with the **current year**, not an error. |
| `GET /account/stats/year/{year}` | Same free-account behavior: any other year comes back as the current year, `200`. |
| `GET /account/stats/watched-time` | Total minutes, movies vs episodes, plus unique title counts. |
| `GET /account/following` | Approved follows only. |
| `GET /account/following/requests` | Requests you sent. |
| `GET /account/followers` | |
| `GET /account/followers/requests` | Pending requests you received. Private accounts. |
| `PUT /account/followers/requests/{follow_id}/{action}` | Approve or deny. Only requests addressed to you, and only while pending. |
| `DELETE /account/followers/{user_id}` | Remove a follower without blocking them. |
| `GET /account/friends` | Mutual approved follows. |
| `GET /account/friends/feed` | Activity of those friends. |
| `GET /account/blocked` | Users you blocked. |

Follow, block, and the friends feed are the account-side pair of the `/users/{id}/follow` and `/block` routes. Use the account routes for "my" lists and the user routes to act on a specific profile.
