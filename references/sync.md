# Sync, tracking, and scrobble

Docs: https://api.wetrakr.com/#/sync and https://api.wetrakr.com/#/scrobble

Every route below is auth `oauth`. Exact query and status codes: `references/endpoints/sync.md`.

## Incremental sync

Do this. Do not walk every list on a timer.

1. `GET /sync/last_activities`. Each section is a last-changes object (timestamps such as `last_tracking_watched_at` and `last_tracking_removed_at`). Compare with what you stored. If nothing moved, stop.
2. If a stamp is newer than `journal_visible_until`, the journal row is not visible yet. Stamps move at write time. Rows appear about **5 seconds** later. Read that section on the next pass.
3. `GET /sync/journal?from_date=<previous mark>`. Required. ISO 8601 or Unix timestamp. Entries are **strictly after** `from_date`, **oldest first**. `limit` default 100, max 1,000. Optional `category` is a comma list: `watched`, `watching`, `waiting`, `planning`, `dropped`, `paused`, `ignored`, `ratings`, `favorites`, `notes`, `likes`, `comments`, `lists`.
4. Apply the journal. It names which titles changed and how, so you do not refetch a whole list. Status moves of one title less than a second apart are stored as the net change. Caught-up shows enter and leave `watched` as `watched` added/removed with **no** `play_id`.
5. Refetch a full section only when the journal is not enough (first sync, or `400` `JOURNAL_EXPIRED`). The error includes `oldest`. Do a full sync, then journal forward from the new mark.
6. Store the new stamps only after the rows you needed were visible.

`from_date` on a single list returns rows changed or removed since that instant. A deleted play does **not** appear in `GET /sync/tracking/watched/history/{target}`. It moves `last_tracking_removed_at`. When that stamp moves, refetch.

Compact first sync: `compact=true` and follow `after` from `X-Pagination-Next`. See `references/conventions.md`. Then fetch detail only for rows you will render.

## Tracking lists

`GET /sync/tracking/{status}/{target}`

| Status | Meaning | Who writes it |
|---|---|---|
| `planning` | Plans to watch. | Client. |
| `watching` | Show in progress. | Client, and WeTrakr when some but not all aired episodes are watched. |
| `waiting` | Caught up, still airing. | WeTrakr. |
| `watched` | Watched. | Client. |
| `paused` | On hold. | Client. Shows. |
| `dropped` | Will not continue. | Client. |
| `playing` | Playing or paused mid-playback, with progress. | Scrobble only. Read-only. |

Allowed targets:

| Media | Statuses you may set |
|---|---|
| Movie | `planning`, `watched`, `dropped` |
| Show | `planning`, `watching`, `watched`, `paused`, `dropped` |
| Season | `watched` |
| Episode | `watched` |

A wrong target for that list is `400`. Unknown status is `404`.

What each list adds on top of the compact media object:

| List | Extra fields |
|---|---|
| `watching`, `paused` | Compact show, `current_season_poster_path` |
| `watched` movie or show | `watched_at`, `watched_count` |
| `watched` episode | `season_number`, `number`, `season_poster_path`, `watched_at`, `watched_count`, `show` `{ id, title, ids }` |
| `planning`, `dropped`, `waiting` | Compact movie or show, `release_date_country` |
| `playing` | Compact movie or episode, `playback` with `status` `playing` or `paused`, `progress_percent`, `runtime_seconds`, `tracked_at` |

`watched` counters: `all` is movies plus episodes. Shows and seasons have their own counts. `GET /sync/tracking/{status}/total-time` is minutes actually watched (rewatches included) on `watched`. On `watching`, `waiting`, and `paused` it is the aired runtime of those shows, not how much the user has seen. `target=seasons` is `400`. Use episodes.

If the user enables "Show caught-up shows in Watched", `waiting` shows also appear in the watched list, its counters, and its total time. The show stays `waiting`. The journal records that as watched added/removed without a `play_id`.

### Automatic transitions

- Adding a show to `watched` marks its episodes watched.
- Adding a season to `watched` marks episodes in that season watched (aired episodes, on the season watch-all route).
- Watching an episode while others remain moves the show to `watching`, or to `waiting` if nothing else has aired.
- Watching the last aired episode moves the show to `waiting`, or to `watched` if the show has ended.
- Removing an episode from a `watched` show moves the show back to `watching`.

Moving an item to a new list removes it from the previous one. Writing a status the title already has is a no-op and returns under `existing`, not `added`.

`POST /sync/tracking` returns `added.plays`: each created play has `play_id`. Delete that one play with `DELETE /sync/tracklogs/{play_id}` (`_track_log_id` in history is the same id).

## Write body

`POST /sync/tracking`, `/remove`, `/remove/last`, `/remove/all` share this document. Ratings, favorites, and notes use the same id arrays. Their extra fields are called out below.

```json
{
  "movies": [
    {
      "ids": { "imdb": "tt0088763" },
      "status": "watched",
      "tracked_at": "2026-09-10T21:14:00.000Z"
    }
  ],
  "shows": [
    { "id": 1391953, "status": "watching" },
    {
      "ids": { "tmdb": 1399 },
      "status": "watched",
      "seasons": [
        { "number": 1, "status": "watched" },
        {
          "number": 2,
          "episodes": [
            { "number": 1, "status": "watched" },
            { "number": 2, "status": "watched" }
          ]
        }
      ]
    }
  ],
  "seasons": [{ "id": 135905, "status": "watched" }],
  "episodes": [{ "id": 174658, "status": "watched", "use_release_date": true }]
}
```

Identity rules:

- `id` is the WeTrakr id, never a TMDB id. Send TMDB as `ids.tmdb`. Also `ids.imdb` and `ids.tvdb`.
- The two id spaces overlap. An item whose type does not match the array it was sent in comes back in `notFound`.
- Every item needs a `status` on add and on every remove. An episode inherits its season's status, then its show's. A season inherits the show's. Missing status comes back in `errored`.
- `status: "none"` clears a status and does **not** delete plays. On a watched title it is `errored`. Use remove for plays.
- With nested `seasons`, the nested items are the scope. A show with `status: "watched"` and nested seasons does **not** mark the whole show watched. A season that lists `episodes` is not itself changed. Remove behaves the same: nested seasons do not remove the whole show.
- Top-level `seasons` and `episodes` on add accept only `watched`. A season id marks each aired episode.

Watch dates, on `watched` only. Default is now.

- `tracked_at` — ISO timestamp for that play.
- `use_release_date: true` — use the title's release or air date.
- `tracked_at_unknown: true` — the user watched it and does not know when.

These three also work on nested episodes and seasons, including when set on the season or the show (1.0.7). They used to be dropped and the play was dated now.

Remove:

- `POST /sync/tracking/remove` — drops the list membership named by `status`. For `watched`, `tracked_at` removes that play. Without it, the latest play goes, so a title watched twice stays watched.
- `POST /sync/tracking/remove/last` — only the most recent play, then the previous play becomes current.
- `POST /sync/tracking/remove/all` — every play, including rewatches, so the title leaves history. A whole show with `status: "watched"` and no nested seasons clears every episode.
- Ids that do not exist come back in `notFound`. Items with nothing to remove come back in `errored`. An empty `removed` is not success.

`POST /sync/tracking/update` moves one existing play. Each item has `status`, which play (`play_id`, or current `tracked_at` to the second), and `updated_tracked_at`. No match: the item is `errored` and nothing changes.

History vs the watched list: the list is one row per title (latest play, plus `watched_count`). `GET /sync/tracking/watched/history/{target}` is one row per play, `target` `movies` or `episodes` only.

Other structural routes (bodies are on the endpoint page): season watch-all / unwatch-all / ignore-all / rewatch, episode watch-up-to / ignore / ignore-previous, show unwatch-all (destroys history, cannot be undone), unwatch-last (one pass), rewatch-restart, rewatch-from-episode (not from a special), `PUT /sync/tracking/show/{show_id}/specials` with boolean `track_specials` (season 0 counts toward progress; it does not mark anything by itself).

Ignored episodes are resolved for "next episode" and are not watched. A watched episode is never downgraded to ignored. Ignore is idempotent.

`GET /sync/tracking/episode/{id}/previous-unwatched-count` only counts. It does not mark anything.

## Ratings, favorites, notes, likes

Ratings: `POST /sync/ratings` with the same id arrays plus `rating` from 0 to 10, one decimal. Rating a show does not rate its seasons. Rating again replaces. A bad number is `errored` and the rest of the batch is saved. Read at `interactions.user.rating` (`rating`, `rated_at`). `POST /sync/ratings/remove` needs ids only.

Favorites: `POST /sync/favorites`. Optional short reason (`why` in the docs). People are allowed. Remove uses the same ids.

Notes: `POST /sync/notes`. Always private. There is no share setting. An imported `privacy` value changes nothing. `text` is required, max 10,000 characters. Empty text is `errored` for that item. Free accounts: 100 notes, then `420`. The 420 copy mentions 1,000 for VIP. That VIP cap is not enforced. Notes embed the title ids. Seasons and episodes embed the show ids. `spoiler` may be present on the note object.

Likes: `GET /sync/likes/{target}` is pointers (list id and name, or comment id and text), not the full object. Newest first.

Journal rows for ratings and notes include the current value: `rating`, or `note` `{ text, spoiler }`.

## Scrobble

Body for start, pause, stop, and optional cancel. Send **either** `movie` **or** `show` plus `episode`, never both.

```json
{
  "movie": {
    "title": "The Dark Knight",
    "year": 2008,
    "ids": { "tmdb": 155, "imdb": "tt0468569" }
  },
  "progress": 12.5,
  "app_version": "1.4.0"
}
```

Episode form: `show` (WeTrakr id, or title, year, and ids) and `episode` (`season` + `number`, or a WeTrakr episode id, in which case `show` may be omitted). `progress` is 0 to 100 and required on start. The response echoes the title WeTrakr resolved. A bad season/episode match shows up there. Unknown title or episode is `404`.

| Call | Effect |
|---|---|
| `POST /scrobble/start` | Opens or seeks. `201`. A second start on the same item only moves `progress`. Starting a different title closes the previous session (watched if its runtime was up, otherwise paused). |
| `POST /scrobble/pause` | Stays in now playing. `200`. |
| `POST /scrobble/stop` | `>= 80` marks watched: `201`, `action: "scrobble"`. Below 80: `200`, `action: "pause"`, session stays so it can resume. A stop at `>= 80` works without a prior start. A play already logged for the same title less than one runtime ago (15 minutes to 4 hours, 30 minutes if runtime is unknown) is not logged again. The answer is still `201`. A scrobble rewatch counts only after that window. |
| `POST /scrobble/checkin` | Start without a reliable stop. Progress is optional (default 0). WeTrakr advances from the clock and marks watched within a minute of the runtime elapsing. Unknown runtime stays `playing` until a stop. |
| `GET /scrobble/playing` | Current session for resume. Nothing playing: `{ "status": "none" }`. Otherwise `status` is `playing` or `paused`, plus `target` `movie` or `episode` and `progress_percent`. |
| `DELETE /scrobble/playing` | Drops the session and does not log a play. No body cancels the latest session. A movie or show+episode body cancels that title. The title returns to its previous state. `404` if nothing matches. |

The old 5% threshold, watched lock, and abandoned-session rule are gone. A start at 0% or a pause on a watched title opens a rewatch session.
