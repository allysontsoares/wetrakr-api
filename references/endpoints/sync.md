# sync endpoints

Extracted from the WeTrakr docs app on 2026-10-01 (changelog 1.0.7). Each route is `https://api.wetrakr.com` plus the path. Send `wetrakr-api-version: 1`.

Account sync and scrobble. Algorithms, tracking lists, and body shapes: `references/sync.md`.

Auth `oauth` requires `Authorization: Bearer`. `apiKey` requires `wetrakr-api-key`. `optionalOauth` works with the key; a token adds the caller's state. An empty auth on a direct object means the symbols line is the source.

## Index

- `POST /scrobble/checkin` — Check in
- `POST /scrobble/pause` — Pause playback
- `DELETE /scrobble/playing` — Cancel playback
- `GET /scrobble/playing` — What is playing now
- `POST /scrobble/start` — Start playback
- `POST /scrobble/stop` — Stop playback
- `GET /sync/favorites/{target}` — Get your favorites
- `GET /sync/favorites/counters` — Count your favorites
- `POST /sync/favorites/remove` — Remove favorites
- `POST /sync/favorites` — Add favorites
- `GET /sync/journal` — Get the journal
- `GET /sync/last_activities` — Get last activities
- `GET /sync/likes/{target}` — Get your likes
- `GET /sync/notes/{target}` — Get your notes
- `GET /sync/notes/counters` — Count your notes
- `POST /sync/notes/remove` — Remove notes
- `POST /sync/notes` — Write notes
- `GET /sync/ratings/{target}` — Get your ratings
- `GET /sync/ratings/counters` — Count your ratings
- `POST /sync/ratings/remove` — Remove ratings
- `POST /sync/ratings` — Rate items
- `GET /sync/tracking/{status}/{target}` — Get a tracking list
- `GET /sync/tracking/{status}/counters` — Count a tracking list
- `GET /sync/tracking/{status}/total-time` — Get time spent on a list
- `GET /sync/tracking/episode/{id}/previous-unwatched-count` — Count what is left before an episode
- `POST /sync/tracking/episode/ignore-previous` — Ignore what came before
- `POST /sync/tracking/episode/ignore` — Ignore one episode
- `POST /sync/tracking/episode/watch-up-to` — Watch up to an episode
- `POST /sync/tracking/remove/all` — Remove every play
- `POST /sync/tracking/remove/last` — Undo the latest play
- `POST /sync/tracking/remove` — Remove items from a tracking list
- `POST /sync/tracking/season/ignore-all` — Ignore a whole season
- `POST /sync/tracking/season/rewatch` — Rewatch a season
- `POST /sync/tracking/season/unwatch-all` — Unwatch a whole season
- `POST /sync/tracking/season/watch-all` — Watch a whole season
- `PUT /sync/tracking/show/{show_id}/specials` — Count specials or not
- `POST /sync/tracking/show/rewatch-from-episode` — Rewatch from an episode
- `POST /sync/tracking/show/rewatch-restart` — Rewatch a show from the start
- `POST /sync/tracking/show/unwatch-all-episodes` — Wipe the history of a show
- `POST /sync/tracking/show/unwatch-last-episodes` — Undo the latest pass
- `POST /sync/tracking/update` — Change a play date
- `GET /sync/tracking/watched/history/{target}` — Get the watch history
- `POST /sync/tracking` — Add items to a tracking list
- `DELETE /sync/tracklogs/{id}` — Delete one play from the history

## POST /scrobble/checkin

- Docs id: `scrobble-checkin`
- Auth: `oauth`

Say the user started watching something, in a single call and without progress. For an app that knows when playback begins but cannot follow it or be sure a stop will arrive: WeTrakr works the progress out from the clock, and the title is marked as watched within a minute of its runtime being up. A title whose runtime WeTrakr does not know stays in playing until a stop arrives. Sending progress is allowed and sets the starting point.

### Body

- `progress` (number, optional) — Where playback starts, 0 to 100. Optional, default 0.

### Responses

- **201** — Check-in registered, with what WeTrakr resolved.

## POST /scrobble/pause

- Docs id: `scrobble-pause`
- Auth: `oauth`

The user paused. The item stays in now playing with the current progress.

Body shape is the scrobble item in `references/sync.md`.

### Responses

- **200** — Pause registered.

## DELETE /scrobble/playing

- Docs id: `scrobble-cancel`
- Auth: `oauth`

Drop a playback session without logging a play: the user hit play by mistake, or your app is not sure the stop arrived. Without a body it cancels the latest session; with movie, or show plus episode (same shape as start), the session of that title. The title goes back to where it was before the session (a rewatch returns to watched with its date) and, if it had no state, leaves the playing list.

Body shape is the scrobble item in `references/sync.md`.

### Responses

- **200** — Session cancelled.
- **404** — Nothing is playing, or that title has no session.

## GET /scrobble/playing

- Docs id: `scrobble-playing`
- Auth: `oauth`

What the user is watching right now, with the title resolved and the progress as it stands. Call it when your app opens to pick up a session started somewhere else, instead of guessing. When nothing is playing the body is {"status":"none"}.

### Responses

- **200** — The current playback.

## POST /scrobble/start

- Docs id: `scrobble-start`
- Auth: `oauth`

Tell WeTrakr the user started, resumed or seeked. The item shows up as now playing with the given progress, and calling it again on something already playing just moves the position, which is how a seek is reported. Only one thing plays at a time: starting another title closes the previous session (marked watched if its runtime was up, paused otherwise). The response echoes the title and episode WeTrakr resolved, so a wrong match (absolute against seasonal numbering) shows at once; an unknown title or episode answers 404.

Body shape is the scrobble item in `references/sync.md`.

### Responses

- **201** — Playback registered, with what WeTrakr resolved.

## POST /scrobble/stop

- Docs id: `scrobble-stop`
- Auth: `oauth`

The user stopped. At 80 percent or more the title is marked as watched (a scrobble, 201 action scrobble); below that it is treated as a pause (200 action pause) and the session stays paused so the user can pick it up later. A stop at 80 percent or more works without a start too: it logs the play. A play already logged for the same title less than one runtime ago (15 minutes to 4 hours, 30 minutes when the runtime is unknown) is not logged again and the answer is the same 201, so a rewatch through scrobble counts once that window has passed.

Body shape is the scrobble item in `references/sync.md`.

### Responses

- **201** — Marked as watched.
- **200** — Below the threshold: kept as paused.

## GET /sync/favorites/{target}

- Docs id: `favorites-list`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`, `sync`

Favorites of the user for one target, most recent first. Entries are the media objects with your favorite state under interactions.favorite. Supports from_date for incremental sync.

### Path params

- `target` (string, required) — movies, shows, seasons, episodes, people or all.

### Query

- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100 (100 by default and 5,000 at most with compact=true).
- `extended` (string, optional) — Field sets to add to each media object.
- `from_date` (datetime, optional) — Only entries updated after this ISO date, including removed ones.
- `compact` (boolean, optional) — true for small rows (type, id, ids and what the list adds) and up to 5,000 per page, built for syncing. See Compact pages and cursors in Conventions. extended and sort_by are ignored.
- `after` (string, optional) — compact=true only: the X-Pagination-Next value of the previous page, to get the next one. Compact pages are walked only with this cursor (page answers 400 CURSOR_REQUIRED), and after without compact=true answers 400 CURSOR_NEEDS_COMPACT: see Compact pages and cursors.

### Responses

- **200** — Array of favorite entries. Paging state comes in the X-Pagination-* headers.

## GET /sync/favorites/counters

- Docs id: `favorites-counters`
- Auth: `oauth`

How many favorites the user has per target.

### Responses

- **200** — Counters per target.

## POST /sync/favorites/remove

- Docs id: `favorites-remove`
- Auth: `oauth`

Unmark the given items. Same body as adding.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per target.

## POST /sync/favorites

- Docs id: `favorites-add`
- Auth: `oauth`

Mark movies, shows, seasons, episodes or people as favorites, optionally with a short why.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per target plus unresolved items.

## GET /sync/journal

- Docs id: `sync-journal`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`, `sync`

Every title that changed since from_date, oldest first: the step after last activities. Last activities says which sections moved; the journal says which titles, and how, so you can update your copy without refetching whole lists.

### Query

- `from_date` (string, required) — Your last sync mark, ISO 8601 or a Unix timestamp. Entries strictly after it come back.
- `category` (string, optional) — Only these categories, comma-separated: watched, watching, waiting, planning, dropped, paused, ignored, ratings, favorites, notes, likes, comments, lists.
- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Entries per page, default 100, maximum 1000. The journal keeps its own default: send limit to be explicit.

### Responses

- **200** — The changes, oldest first. Paging state comes in the X-Pagination-* headers.
- **400** — from_date is missing, does not parse, or is older than the journal keeps (error JOURNAL_EXPIRED, with oldest: do a full sync and use the journal from then on). Also an unknown category.

## GET /sync/last_activities

- Docs id: `sync-last-activities`
- Auth: `oauth`

Send the user here to sign in and approve your app. This is a browser redirect, not a JSON endpoint. WeTrakr renders its own login page. On success the user is redirected back to your redirect_uri with a one-time authorization code.

### Responses

- **200** — One last changes object per section.

## GET /sync/likes/{target}

- Docs id: `account-likes`
- Auth: `oauth`

What the user liked, newest first: lists, and comments and replies. Each entry is a pointer plus the date, not the object itself: the list arrives as its id and name, the comment as its id and text, and the rest you ask for where it lives.

### Query

- `from_date` (string, optional) — Only what changed since this date, for incremental sync. ISO 8601 or a Unix timestamp. See Incremental sync.

### Responses

- **200** — Array of likes. Each one carries either list or comment, never both.

## GET /sync/notes/{target}

- Docs id: `account-notes`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`

The user's notes for one target, newest first, 20 per page, each embedding the item it is about with its ids (and, for seasons and episodes, its show with the show's ids), so you can match a note to its title without fetching it.

### Query

- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `from_date` (string, optional) — Only what changed since this date, for incremental sync. ISO 8601 or a Unix timestamp. See Incremental sync.

### Responses

- **200** — A page of note objects with X-Pagination-* headers.

## GET /sync/notes/counters

- Docs id: `account-notes-counters`
- Auth: `oauth`

How many notes the user has per target.

### Responses

- **200** — Counters per target.

## POST /sync/notes/remove

- Docs id: `account-notes-remove`
- Auth: `oauth`

Delete the notes on the given items. Same body as writing, only the ids matter.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per type.

## POST /sync/notes

- Docs id: `account-notes-add`
- Auth: `oauth`

Attach a personal note to titles or people. A note is always private: there is no setting to share it, and nobody but its author ever reads it (a privacy field on older imported notes, such as friends, changes nothing). Free accounts can keep 100 notes. The text is required and takes up to 10,000 characters: an empty or blank note, or a longer one, comes back in errored and the rest are saved. The 420 message quotes a VIP ceiling of 1,000, but there is no tier above VIP yet and that ceiling is not enforced.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per type plus unresolved items.
- **420** — Note quota reached.

## GET /sync/ratings/{target}

- Docs id: `ratings-list`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`, `sync`

Ratings given by the user for one target, most recent first. Entries are the media objects with your rating under interactions.user.rating: the value in rating (0 to 10, one decimal) and the date in rated_at. With from_date only ratings changed or removed since that date come back, which is what a sync wants.

### Path params

- `target` (string, required) — movies, shows, seasons, episodes, people or all.

### Query

- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page: 20 by default, maximum 100 (100 by default and 5,000 at most with compact=true).
- `extended` (string, optional) — Field sets to add to each media object.
- `from_date` (datetime, optional) — Only entries updated after this ISO date, including removed ones.
- `compact` (boolean, optional) — true for small rows (type, id, ids and what the list adds) and up to 5,000 per page, built for syncing. See Compact pages and cursors in Conventions. extended and sort_by are ignored.
- `after` (string, optional) — compact=true only: the X-Pagination-Next value of the previous page, to get the next one. Compact pages are walked only with this cursor (page answers 400 CURSOR_REQUIRED), and after without compact=true answers 400 CURSOR_NEEDS_COMPACT: see Compact pages and cursors.

### Responses

- **200** — compact=true: one small row per rating. X-Pagination-Next carries the cursor for the next page (send it as after) while there are more.
- **200** — Array of rating entries. Paging state comes in the X-Pagination-* headers.
- **404** — Unknown target.

## GET /sync/ratings/counters

- Docs id: `ratings-counters`
- Auth: `oauth`

How many ratings the user has per target.

### Responses

- **200** — Counters per target.

## POST /sync/ratings/remove

- Docs id: `ratings-remove`
- Auth: `oauth`

Delete ratings for the given items. Same body as rating, the rating value is not needed.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per target.

## POST /sync/ratings

- Docs id: `ratings-add`
- Auth: `oauth`

Rate movies, shows, seasons, episodes or people, many at once. Rating a show does not rate its seasons; list them explicitly. Rating again replaces the previous value. A rating that is not a number from 0 to 10 comes back in errored and the rest of the batch is saved.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per target plus unresolved items.

## GET /sync/tracking/{status}/{target}

- Docs id: `tracking-list`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`

Items in one tracking list. Each entry is the compact media object plus the fields of that list, and nothing else: the tables below show what each list adds and what extended unlocks.

### Path params

- `status` (string, required) — Tracking list: watching, waiting, watched, planning, dropped, paused or playing.
- `target` (string, required) — movies, shows, seasons, episodes or all. Each list only allows some targets, see the table above.

### Query

- `page` (integer, optional) — Page number, starting at 1. Not with compact=true, which uses after.
- `limit` (integer, optional) — Items per page: 20 by default, maximum 100 (100 by default and 5,000 at most with compact=true). Walk the pages with page and the X-Pagination-* headers.
- `extended` (string, optional) — Field sets to add to each entry. interactions, episode_level_1 and show_level_1 unlock the extras listed in the table below.
- `sort_by` (string, optional) — added (default, most recent first), my_last_watched, my_last_played, title, avg_rating, release_date, votes, runtime, people_watched.
- `sort_dir` (string, optional) — desc (default) or asc.
- `title_search` (string, optional) — Narrow the list to titles matching this text.
- `watched_from` (date, optional) — Watched list only: watched on or after this day (YYYY-MM-DD). Filters by when the user watched it, which is not the same as from_date.
- `watched_to` (date, optional) — Watched list only: watched on or before this day.
- `from_date` (string, optional) — Only the entries whose row changed since this date, for incremental sync. ISO 8601 or a Unix timestamp. See Incremental sync.
- `compact` (boolean, optional) — true for small rows (type, id, ids and what the list adds) and up to 5,000 per page, built for syncing. See Compact pages and cursors in Conventions. extended and sort_by are ignored.
- `after` (string, optional) — compact=true only: the X-Pagination-Next value of the previous page, to get the next one. Compact pages are walked only with this cursor (page answers 400 CURSOR_REQUIRED), and after without compact=true answers 400 CURSOR_NEEDS_COMPACT: see Compact pages and cursors.

### Responses

- **200** — compact=true on planning: one small row per title. On watched each row carries last_watched_at and plays (movies, shows and episodes; seasons have no plays of their own) instead of status and tracked_at.
- **200** — Array of entries: the compact media object plus the fields of that list. The example is one entry of watching/shows; the rows marked below are what the other lists and targets add. Paging state comes in the X-Pagination-* headers.
- **400** — The target is not allowed for that list.
- **404** — Unknown status.

## GET /sync/tracking/{status}/counters

- Docs id: `tracking-counters`
- Auth: `oauth`

How many items the list holds per target. For watched, all is movies plus episodes, since shows and seasons have their own tabs.

### Path params

- `status` (string, required) — Tracking list: watching, waiting, watched, planning, dropped, paused or playing.

### Responses

- **200** — Counters per target.

## GET /sync/tracking/{status}/total-time

- Docs id: `tracking-total-time`
- Auth: `oauth`

Minutes of runtime and number of plays behind a tracking list. On watched it is time actually watched, rewatches included: use it for "you have watched 41 days of TV" style stats. On the show lists (watching, waiting, paused) it is the aired runtime of those shows, watched or not: how much the list holds, not how much of it the user has seen.

### Path params

- `status` (string, required) — Tracking list: watching, waiting, watched, planning, dropped, paused or playing.

### Query

- `target` (string, optional) — movies, shows, episodes or all (default). Seasons have no plays of their own: count their episodes.

### Responses

- **200** — Totals.
- **400** — Unknown target, or seasons.

## GET /sync/tracking/episode/{id}/previous-unwatched-count

- Docs id: `tracking-episode-previous-unwatched`
- Auth: `oauth`

How many earlier episodes of the show the user has neither watched nor ignored. Useful before offering "watch everything up to here". Reads only, changes nothing.

### Path params

- `id` (integer, required) — WeTrakr episode id.

### Responses

- **200** — The counts.

## POST /sync/tracking/episode/ignore-previous

- Docs id: `tracking-episode-ignore-previous`
- Auth: `oauth`

Mark every episode earlier than this one as ignored, leaving the episode itself alone. For the user who starts a show halfway and is not going back. Episodes already watched or ignored are left as they are, so calling it twice changes nothing.

### Body

- `id` (integer, required) — WeTrakr episode id.

### Responses

- **200** — What was ignored.

## POST /sync/tracking/episode/ignore

- Docs id: `tracking-episode-ignore`
- Auth: `oauth`

Mark a single episode as ignored: it stops being offered as the next one and counts as resolved, without counting as watched. An episode already watched is never downgraded.

### Body

- `id` (integer, required) — WeTrakr episode id.

### Responses

- **200** — The result.

## POST /sync/tracking/episode/watch-up-to

- Docs id: `tracking-episode-watch-up-to`
- Auth: `oauth`

Mark an episode and every earlier one of the show as watched. Episodes already watched are skipped so no rewatch is created, except the one you point at, which is always marked. Ignored episodes are marked too.

### Body

- `tracked_at_unknown` (boolean, optional) — The user saw them but does not know when.
- `use_release_date` (boolean, optional) — Date every episode on its own air date.
- `tracked_at` (datetime, optional) — One ISO date for all of them. Without any of these three, now.
- `id` (integer, required) — WeTrakr episode id.

### Responses

- **200** — What was marked.

## POST /sync/tracking/remove/all

- Docs id: `tracking-remove-all`
- Auth: `oauth`

Remove all plays of each item in the given list, including rewatches, so they leave the watch history. Same body as add: a movie or episode by id or external ids, one episode or a whole season with the nested form ({ shows: [{ ids: { tmdb }, status: 'watched', seasons: [{ number, episodes: [{ number }] }] }] }), or a whole show with status watched, which clears every episode. Every item needs a status (its own or its parent's): without one it comes back in errored. With nested seasons only the nested items are removed, never the whole show, even with status watched on the show. last_activities moves the removed stamp of each target.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — What was removed: titles per list, and episode plays for episodes.
- **200** — Nothing was removed: ids that do not exist come back in notFound, and items that exist but had nothing with that status in errored. An empty answer never means success.

## POST /sync/tracking/remove/last

- Docs id: `tracking-remove-last`
- Auth: `oauth`

Remove only the most recent play of each item and fall back to the previous one. Handy for "oops, I did not watch that" from a media center. Like remove, ids that do not exist come back in notFound, items without status or with nothing to remove in errored, and currentTracks lists only the items that lost a play.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per list and target.

## POST /sync/tracking/remove

- Docs id: `tracking-remove`
- Auth: `oauth`

Take items out of the list named by their status. Same body as add, including seasons and the nested show > seasons > episodes form by number. For watched items with tracked_at, only that play is removed; without it the latest play goes, so a title watched twice stays watched: use remove/all to clear every play.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per list and target.

## POST /sync/tracking/season/ignore-all

- Docs id: `tracking-season-ignore-all`
- Auth: `oauth`

Mark every aired episode of one or more seasons as ignored: they stop being offered as the next episode and count as resolved, without counting as watched. Episodes already watched are never downgraded, so calling it twice changes nothing.

### Responses

- **200** — What was ignored.

## POST /sync/tracking/season/rewatch

- Docs id: `tracking-season-rewatch`
- Auth: `oauth`

Start a season again: its episodes go back to unwatched and the show points at the first one. The history is kept, so the plays already logged stay in the watched list and in the counters.

### Body

- `season_id` (integer, required) — WeTrakr season id.

### Responses

- **200** — The new state of the season.
- **400** — The season id is not a positive integer.

## POST /sync/tracking/season/unwatch-all

- Docs id: `tracking-season-unwatch-all`
- Auth: `oauth`

Remove the watched history of every episode of one or more seasons, aired or not, and leave the season untracked. Only touches watched plays: nothing else about the title changes.

### Responses

- **200** — What was removed.

## POST /sync/tracking/season/watch-all

- Docs id: `tracking-season-watch-all`
- Auth: `oauth`

Mark every aired episode of one or more seasons as watched, in a single call. Episodes that have not aired are left alone, and the season itself becomes watched once they are all in. A season that fails does not stop the rest: the failures come back under errors. seasons counts the seasons this call completed, and episodes the plays it added: a season that was already watched answers seasons 0 and episodes 0, and nothing changes.

### Responses

- **200** — What was marked.

## PUT /sync/tracking/show/{show_id}/specials

- Docs id: `tracking-show-specials`
- Auth: `oauth`

Choose whether season 0 counts towards the progress of a show for this user. Turning it on does not mark anything: it only changes how the progress and the next episode are worked out, and the show is recalculated right away.

### Path params

- `show_id` (integer, required) — WeTrakr show id.

### Body

- `track_specials` (boolean, required) — true to count the specials of this show, false to leave them out. Must be a real boolean, not a string.

### Responses

- **200** — The preference, as stored.
- **400** — The value is not a boolean.
- **404** — No show with that id.

## POST /sync/tracking/show/rewatch-from-episode

- Docs id: `tracking-show-rewatch-from`
- Auth: `oauth`

Start a show again from a given episode: that one and everything after it go back to unwatched, what came before keeps its state, and the show points at the episode you chose. The history is kept. Specials follow the preference of the show, and you cannot restart from one.

### Body

- `episode_id` (integer, required) — WeTrakr episode id to restart from.

### Responses

- **200** — The new state of the show.
- **400** — You cannot restart from a special.

## POST /sync/tracking/show/rewatch-restart

- Docs id: `tracking-show-rewatch-restart`
- Auth: `oauth`

Start a show again: every episode goes back to unwatched and the show points at the first aired one. The history is kept, so the plays already logged stay in the watched list and in the counters.

### Body

- `id` (integer, required) — WeTrakr show id.

### Responses

- **200** — The new state of the show.

## POST /sync/tracking/show/unwatch-all-episodes

- Docs id: `tracking-show-unwatch-all`
- Auth: `oauth`

Delete every play the user logged for a show: its episodes, its seasons and the show itself go back to untracked, specials included. This removes the history, not just the current state, and cannot be undone.

### Body

- `id` (integer, required) — WeTrakr show id.

### Responses

- **200** — What was removed.

## POST /sync/tracking/show/unwatch-last-episodes

- Docs id: `tracking-show-unwatch-last`
- Auth: `oauth`

Remove the most recent play of every episode of the show, which undoes one pass through it. A show watched twice goes back to watched once; one watched once goes back to unwatched. Specials follow the preference of the show. Calling it again removes another pass.

### Body

- `id` (integer, required) — WeTrakr show id.

### Responses

- **200** — What was removed.

## POST /sync/tracking/update

- Docs id: `tracking-update`
- Auth: `oauth`

Move an existing play to another date. Each item names its status, which play to move (play_id, or its current tracked_at to the second) and the new updated_tracked_at. Only that play moves: when nothing matches, the item comes back in errored and nothing changes.

### Body

- `shows` (array, optional) — Show items, same fields.
- `seasons` (array, optional) — Season items, same fields.
- `episodes` (array, optional) — Episode items, same fields.

### Responses

- **200** — How many plays moved, and the items that matched no play.
- **200** — No play of that item at that tracked_at: nothing changed.

## GET /sync/tracking/watched/history/{target}

- Docs id: `tracking-watched-history`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`

Every play, rewatches included, newest first. The watched list gives each title once with its latest play; this gives one row per play, which is what a history screen or a media center library needs.

### Path params

- `target` (string, required) — movies or episodes.

### Query

- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Plays per page: default 100, maximum 100, or 5,000 with compact=true.
- `watched_from` (date, optional) — Plays on or after this day (YYYY-MM-DD) or moment (ISO 8601).
- `watched_to` (date, optional) — Plays on or before this day or moment.
- `from_date` (string, optional) — Only the plays added or moved since this date, for incremental sync. ISO 8601 or a Unix timestamp. A deleted play does not show up here: it moves last_tracking_removed_at in last activities, so refetch when that moves.
- `extended` (string, optional) — Field sets for the title of each play: movie_level_1, movie_level_2, episode_level_1, episode_level_2.
- `after` (string, optional) — compact=true only: the X-Pagination-Next value of the previous page, to get the next one. Compact pages are walked only with this cursor (page answers 400 CURSOR_REQUIRED), and after without compact=true answers 400 CURSOR_NEEDS_COMPACT: see Compact pages and cursors.

### Responses

- **200** — A page of plays. Paging state comes in the X-Pagination-* headers. The example is an episode play; a movie play carries movie instead of episode.
- **200** — compact=true: one small row per play.
- **400** — The target is not movies or episodes, or a date does not parse.

## POST /sync/tracking

- Docs id: `tracking-add`
- Auth: `oauth`

Save movies, shows, seasons or episodes to a tracking list, many at once. Moving an item to a new list removes it from the previous one. For watched items the date defaults to now: send tracked_at for a specific date, use_release_date: true to use the title's release date, or tracked_at_unknown: true when the date is not known.

Body shape is the shared batch document in `references/sync.md`. The extractor does not inline it on this route.

### Responses

- **200** — Counts per list and target, plus the items that could not be resolved. added.plays lists every play created, with its play_id, so a single play can be removed at once with DELETE /sync/tracklogs/{play_id} without reading the journal first. Writing a status the title already has changes nothing: it comes back under existing instead of added.

## DELETE /sync/tracklogs/{id}

- Docs id: `tracking-delete-play`
- Auth: `oauth`

Delete a single play by its log id, the _track_log_id of an entry in the watch history of the title, and leave the item pointing at whatever play remains.

### Path params

- `id` (string, required) — Play log id, from the watch history of the title.

### Responses

- **200** — Play deleted.
- **404** — No play with that id for this user.
