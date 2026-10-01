# account endpoints

Extracted from the WeTrakr docs app on 2026-10-01 (changelog 1.0.7). Each route is `https://api.wetrakr.com` plus the path. Send `wetrakr-api-version: 1`.

The signed-in account and other users' profiles. Rules: `references/social.md`.

Auth `oauth` requires `Authorization: Bearer`. `apiKey` requires `wetrakr-api-key`. `optionalOauth` works with the key; a token adds the caller's state. An empty auth on a direct object means the symbols line is the source.

## Index

- `GET /account/blocked` — Get blocked users
- `DELETE /account/followers/{user_id}` — Remove a follower
- `PUT /account/followers/requests/{follow_id}/{action}` — Approve or deny a request
- `GET /account/followers/requests` — Get pending requests you received
- `GET /account/followers` — Get followers
- `GET /account/following/requests` — Get pending requests you sent
- `GET /account/following` — Get following
- `GET /account/friends/feed` — Get the friends activity feed
- `GET /account/friends` — Get friends
- `GET /account/last` — Get recent activity
- `GET /account/plan-usage` — Get plan usage
- `GET /account/settings` — Get your account
- `POST /account/settings` — Update your account
- `GET /account/stats/all` — All-time stats
- `GET /account/stats/month` — Monthly stats
- `GET /account/stats/watched-time` — Watched time
- `GET /account/stats/week` — Weekly stats
- `GET /account/stats/year/{year}` — Yearly stats
- `DELETE /users/{id}/block` — Unblock a user
- `POST /users/{id}/block` — Block a user
- `GET /users/{id}/comments/{target}` — Get comments
- `GET /users/{id}/comments/counters` — Count comments
- `GET /users/{id}/favorites/{target}` — Get favorites
- `GET /users/{id}/favorites/counters` — Count favorites
- `DELETE /users/{id}/follow` — Unfollow a user
- `GET /users/{id}/followers` — Get followers
- `GET /users/{id}/following` — Get following
- `POST /users/{id}/follow` — Follow a user
- `GET /users/{id}/likes/{target}` — Get likes
- `GET /users/{id}/lists` — Get lists
- `GET /users/{id}/ratings/{target}` — Get ratings
- `GET /users/{id}/ratings/counters` — Count ratings
- `GET /users/{id}/shows/{show_id}/episode-ratings` — Get episode ratings for a show
- `GET /users/{id}/tracking/{status}/{target}` — Get a tracking list
- `GET /users/{id}/tracking/{status}/counters` — Count a tracking list
- `GET /users/{id}` — Get a profile

## GET /account/blocked

- Docs id: `account-blocked`
- Auth: `oauth`

Users the signed-in user has blocked.

### Responses

- **200** — Array of blocks with the blocked user and date.

## DELETE /account/followers/{user_id}

- Docs id: `account-follower-remove`
- Auth: `oauth`

Make a user stop following you without blocking them.

### Path params

- `user_id` (integer, required) — Id of the follower to remove.

### Responses

- **200** — Removed.
- **404** — That user does not follow you.

## PUT /account/followers/requests/{follow_id}/{action}

- Docs id: `account-follow-resolve`
- Auth: `oauth`

Resolve a pending follow request. Only requests addressed to the signed-in user, and only while pending.

### Path params

- `follow_id` (integer, required) — Id of the pending follow entry.
- `action` (string, required) — approved or denied.

### Responses

- **200** — The updated follow entry.
- **400** — Unknown action.
- **403** — The request is not addressed to you.
- **404** — No pending request with that id.

## GET /account/followers/requests

- Docs id: `account-followers-requests`
- Auth: `oauth`

Follow requests waiting for the user's approval (private accounts only).

### Responses

- **200** — Array of pending follow entries.

## GET /account/followers

- Docs id: `account-followers`
- Auth: `oauth`

Users who follow the signed-in user.

### Query

- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of follow entries.

## GET /account/following/requests

- Docs id: `account-following-requests`
- Auth: `oauth`

Follow requests the user sent that private accounts have not answered yet.

### Responses

- **200** — Array of pending follow entries.

## GET /account/following

- Docs id: `account-following`
- Auth: `oauth`

Users the signed-in user follows, approved follows only.

### Query

- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of follow entries.

## GET /account/friends/feed

- Docs id: `account-friends-feed`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`

What the people the signed-in user follows have been doing, newest first: plays, shows they are watching, ratings, favorites, comments, new lists, likes and follows. Each author's privacy settings apply (a hidden section never shows up, friends-only content only for mutual follows), so it matches what the user would see on WeTrakr. The feed is a fixed window: the 300 newest events across every type and every followed user, the same on every page, with the paging state in X-Pagination-*. total is the number of events in that window, after grouping and privacy. To build it, each source contributes its newest events: plays (up to 600), shows being watched (up to 300), and up to 300 each of ratings, favorites, comments, lists, likes and follows. A show someone keeps watching counts once: one watching event per user and show, the newest. The 300 newest of all of them make the window, so a play older than the 300th event is outside it. Plays logged without a date (imports) and follows that were undone are left out.

### Query

- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Events per page. Default 15, maximum 60.
- `types_in` (string, optional) — Only these event types, comma-separated, in any case: watched, watching, rating, favorite, comment, list, like, follow. An unknown type answers 400 INVALID_PARAMETER.
- `user_id` (integer, optional) — Only the events of this followed user. A value that is not a user id answers 400 INVALID_PARAMETER.

### Responses

- **200** — A page of events and the total.

## GET /account/friends

- Docs id: `account-friends`
- Auth: `oauth`

Friends of the signed-in user: mutual approved follows.

### Responses

- **200** — Array of friends with the date the friendship formed.

## GET /account/last

- Docs id: `account-last`
- Auth: `oauth`

The user's most recent activity across sections: last watched, rated, favorited, commented and listed items, ready for a profile overview.

### Query

- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Recent items per section.

## GET /account/plan-usage

- Docs id: `account-plan-usage`
- Auth: `oauth`
- Symbols: `oauth`

The user's plan, how much of each quota they have used, and the features the plan unlocks. On a free account, read it after sign-in and you can say that the next list will not fit before the user writes it, instead of letting the write come back with a 420.

### Responses

- **200** — Plan, quotas and features.

## GET /account/settings

- Docs id: `account-settings-get`
- Auth: `oauth`

The signed-in user: who they are, where they are, what they can see and the preferences that change how dates and titles should be read. It is a fixed set of fields, not the whole account record, so a new setting in the WeTrakr apps never starts appearing here on its own.

### Responses

- **200** — The account.

## POST /account/settings

- Docs id: `account-settings-update`
- Auth: `oauth`

Change profile, location, privacy, notification and display settings. Send only the fields to change, as nested objects or dotted paths; unknown fields are ignored and the response lists them. Username changes are validated for format and uniqueness. Non-public profile privacy and profile backgrounds need VIP.

### Body

- `networks` (object, optional) — facebook, instagram, tiktok, youtube handles.
- `preferences` (object, optional) — language, date_format (mdy, dmy, ymd), time_24hr, episode_image_mode, include_adult. The WeTrakr apps write more keys here (layouts, sidebars, backgrounds); they are accepted and stored, and they do not come back in the response.
- `streaming_services` (array, optional) — Provider slugs the user subscribes to.

### Responses

- **200** — What was applied, what was ignored, and the account as it stands now.
- **400** — A field failed validation; the body names it.
- **426** — A VIP-only setting on a free account.

## GET /account/stats/all

- Docs id: `account-stats-all`
- Auth: `oauth`
- Symbols: `oauth`

Total time the user has spent watching, split between movies and episodes, with unique title counts.

## GET /account/stats/month

- Docs id: `account-stats-month`
- Auth: `oauth`
- Symbols: `oauth`

Total time the user has spent watching, split between movies and episodes, with unique title counts.

## GET /account/stats/watched-time

- Docs id: `account-stats-watched-time`
- Auth: `oauth`

Total time the user has spent watching, split between movies and episodes, with unique title counts.

### Responses

- **200** — Totals.

## GET /account/stats/week

- Docs id: `account-stats-week`
- Auth: `oauth`
- Symbols: `oauth`

Total time the user has spent watching, split between movies and episodes, with unique title counts.

## GET /account/stats/year/{year}

- Docs id: `account-stats-year`
- Auth: `oauth`
- Symbols: `oauth`

Total time the user has spent watching, split between movies and episodes, with unique title counts.

## DELETE /users/{id}/block

- Docs id: `users-unblock`
- Auth: `optionalOauth`

Lift a block.

### Responses

- **200** — Unblocked.
- **404** — The user was not blocked.

## POST /users/{id}/block

- Docs id: `users-block`
- Auth: `optionalOauth`

Block the user: any follow between you is removed, they cannot follow you again and their comments disappear from what you see.

### Responses

- **201** — Blocked.
- **400** — Trying to block yourself.

## GET /users/{id}/comments/{target}

- Docs id: `users-comments`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `privacy`, `paginated`

Comments the user wrote, newest first, 20 per page. Each comment embeds the compact title it is about.

### Path params

- `id` (integer, required) — WeTrakr user id.

### Query

- `is_long` (boolean, optional) — true for reviews only (200 words or more), false for short comments only. Paging and X-Pagination-Item-Count describe the filtered set; any other value answers 400 INVALID_PARAMETER.
- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.

### Responses

- **200** — Array of comment objects. Paging state comes in the X-Pagination-* headers.

## GET /users/{id}/comments/counters

- Docs id: `users-comments-counters`
- Auth: `optionalOauth`

How many comments the user wrote per target.

### Responses

- **200** — Counters per target.

## GET /users/{id}/favorites/{target}

- Docs id: `users-favorites`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `privacy`, `paginated`, `sync`

The user's favorites for one target, most recent first.

### Path params

- `id` (integer, required) — WeTrakr user id.

### Query

- `from_date` (datetime, optional) — Only entries updated after this ISO date, including removed ones.
- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `compact` (boolean, optional) — true for small rows (type, id, ids and what the list adds) and up to 5,000 per page, built for syncing. See Compact pages and cursors in Conventions. extended and sort_by are ignored.
- `after` (string, optional) — compact=true only: the X-Pagination-Next value of the previous page, to get the next one. Compact pages are walked only with this cursor (page answers 400 CURSOR_REQUIRED), and after without compact=true answers 400 CURSOR_NEEDS_COMPACT: see Compact pages and cursors.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of media objects.

## GET /users/{id}/favorites/counters

- Docs id: `users-favorites-counters`
- Auth: `optionalOauth`

How many favorites the user has per target.

### Responses

- **200** — Counters per target.

## DELETE /users/{id}/follow

- Docs id: `users-unfollow`
- Auth: `optionalOauth`

Stop following the user, or withdraw a pending request.

### Responses

- **200** — Unfollowed.
- **404** — You were not following this user, or already unfollowed it. Nothing changes.

## GET /users/{id}/followers

- Docs id: `users-followers`
- Auth: `apiKey`
- Symbols: `apiKey`

Users who follow this user.

### Query

- `page` (integer, optional) — Page number, starting at 1. With page or limit the list is paged, with X-Pagination-* headers; without them it comes whole, and X-Pagination-Item-Count still gives the total.
- `limit` (integer, optional) — Entries per page, up to 100. Default 20 when paging.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of follow entries.

## GET /users/{id}/following

- Docs id: `users-following`
- Auth: `apiKey`
- Symbols: `apiKey`

Users this user follows, with the date each follow was approved.

### Query

- `page` (integer, optional) — Page number, starting at 1. With page or limit the list is paged, with X-Pagination-* headers; without them it comes whole, and X-Pagination-Item-Count still gives the total.
- `limit` (integer, optional) — Entries per page, up to 100. Default 20 when paging.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of follow entries.

## POST /users/{id}/follow

- Docs id: `users-follow`
- Auth: `optionalOauth`

Follow the user. Public accounts are followed at once; private accounts get a request the owner must approve or deny, and the entry stays pending meanwhile.

### Responses

- **201** — A new follow: approved at once for a public account, pending for a private one.
- **200** — You already follow the user, or already asked: the same entry, unchanged.
- **400** — Trying to follow yourself.
- **403** — The user has blocked you.

## GET /users/{id}/likes/{target}

- Docs id: `users-likes`
- Auth: `optionalOauth`

Lists and comments the user liked.

### Path params

- `id` (integer, required) — WeTrakr user id.

### Responses

- **200** — Array of liked objects.
- **403** — The likes section is private or friends-only.

## GET /users/{id}/lists

- Docs id: `users-lists`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `privacy`, `sync`

Lists the user owns, public ones only unless you are the owner or a friend with access. Each list carries its item counts; extended=preview adds the posters of its first items.

### Query

- `sort` (string, optional) — manual (the owner's order, default), updated, created, name, items.
- `from_date` (datetime, optional) — Only lists updated after this ISO date.
- `extended` (string, optional) — Comma-separated field sets. preview adds preview to each list: up to five of its items (in the list order) with title, ids, poster_path and backdrop_path, enough to draw a list card without reading its items. Empty lists carry none. Other values work as in Extended and append.

### Responses

- **200** — Array of list objects.

## GET /users/{id}/ratings/{target}

- Docs id: `users-ratings`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `privacy`, `paginated`, `sync`

Titles the user has rated, most recent first, each with the score this user gave under user_rating. When the ratings section is not visible to you it answers 403 with error PRIVATE or FRIENDS_ONLY. With your token, interactions.user.rating is still your own score, not theirs.

### Path params

- `id` (integer, required) — WeTrakr user id.

### Query

- `from_date` (datetime, optional) — Only entries updated after this ISO date, including removed ones.
- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `compact` (boolean, optional) — true for small rows (type, id, ids and what the list adds) and up to 5,000 per page, built for syncing. See Compact pages and cursors in Conventions. extended and sort_by are ignored.
- `after` (string, optional) — compact=true only: the X-Pagination-Next value of the previous page, to get the next one. Compact pages are walked only with this cursor (page answers 400 CURSOR_REQUIRED), and after without compact=true answers 400 CURSOR_NEEDS_COMPACT: see Compact pages and cursors.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of media objects. Paging state comes in the X-Pagination-* headers.

## GET /users/{id}/ratings/counters

- Docs id: `users-ratings-counters`
- Auth: `optionalOauth`

How many ratings the user has per target.

### Responses

- **200** — Counters per target.

## GET /users/{id}/shows/{show_id}/episode-ratings

- Docs id: `users-episode-ratings`
- Auth: `optionalOauth`

Every episode rating the user gave within one show, keyed by episode id. Handy for painting a heatmap of a season.

### Path params

- `show_id` (integer, required) — WeTrakr show id.
- `id` (integer, required) — WeTrakr user id.

### Responses

- **200** — Ratings keyed by episode id.
- **200** — Ratings hidden by privacy.
- **400** — The show id is not a number.

## GET /users/{id}/tracking/{status}/{target}

- Docs id: `users-tracking`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `privacy`, `paginated`

One of the user's tracking lists, same shape and parameters as your own lists in Sync. When the tracking section is not visible to you it answers 403 with error PRIVATE or FRIENDS_ONLY.

### Path params

- `status` (string, required) — watching, waiting, watched, planning, dropped or paused.
- `id` (integer, required) — WeTrakr user id.

### Query

- `sort_by` (string, optional) — added (default), my_last_watched, title, avg_rating, release_date, votes, runtime, people_watched.
- `sort_dir` (string, optional) — desc (default) or asc.
- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `compact` (boolean, optional) — true for small rows (type, id, ids and what the list adds) and up to 5,000 per page, built for syncing. See Compact pages and cursors in Conventions. extended and sort_by are ignored.
- `after` (string, optional) — compact=true only: the X-Pagination-Next value of the previous page, to get the next one. Compact pages are walked only with this cursor (page answers 400 CURSOR_REQUIRED), and after without compact=true answers 400 CURSOR_NEEDS_COMPACT: see Compact pages and cursors.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of media objects, without interactions unless you ask for them with extended=interactions. Paging state comes in the X-Pagination-* headers.

## GET /users/{id}/tracking/{status}/counters

- Docs id: `users-tracking-counters`
- Auth: `optionalOauth`

How many items the user has in a list, per target. When the section is not visible to you it answers 403 with error PRIVATE or FRIENDS_ONLY.

### Path params

- `status` (string, required) — watching, waiting, watched, planning, dropped or paused.
- `id` (integer, required) — WeTrakr user id.

### Responses

- **200** — Counters per target.

## GET /users/{id}

- Docs id: `users-profile`
- Auth: `optionalOauth`

Public profile of a user: identity, images, social links, location, plan, activity level and counters for their content. What the user has configured in their account (their privacy settings, their display preferences) is not part of their profile and does not come back. When the profile is private, or friends-only and you are not a friend, only id, username, display name and images come back. Profile privacy and each section's privacy (ratings, favorites, lists...) are separate settings: a private profile can still have public ratings, which the ratings routes serve. Each counter follows its section's privacy, and the list count only counts the lists you can see.

### Responses

- **200** — The public profile.
- **200** — Private profile: the minimal version.
