# Conventions

Host: `https://api.wetrakr.com`. HTTPS only. Resources live at the root. Docs: https://api.wetrakr.com/#/conventions

## Required headers

| Header | Value | When |
|---|---|---|
| `wetrakr-api-version` | `1` | Every request. |
| `wetrakr-api-key` | App client id | Every request except where a route is auth `none` and you are only exchanging a token. A missing key on a protected read is `401` `MISSING_API_KEY`. |
| `Authorization` | `Bearer <access_token>` | Auth `oauth`. Also on `optionalOauth` when you need the caller's tracking, rating, note, favorite, or hidden comments. |
| `Content-Type` | `application/json` | Requests with a body. |
| `wetrakr-api-language` | BCP-47, such as `pt-BR` or `es-ES` | Optional. Title, overview, tagline, and biography follow it when a translation exists. |
| `wetrakr-api-country` | ISO 3166-1 alpha-2, such as `BR` | Optional. Release dates, certifications, and calendar region fall back to this, then the user's country. |
| `Idempotency-Key` | Caller-generated token | Optional on writes. The same key and body replay the first answer. The same key with a different body is `422` `IDEMPOTENCY_KEY_REUSED`. `409` and `429` are not stored, so those may be retried. |

## Auth symbols on a route

- `none` — no user token. OAuth token endpoints.
- `apiKey` — client id only.
- `oauth` — user access token required.
- `optionalOauth` — client id is enough. A token adds `interactions` for that user.
- `paginated` — `page` / `limit` or a cursor, plus `X-Pagination-*`.
- `sync` — supports `from_date` for incremental sync.
- `vip` — without VIP the call is `426` `VIP_REQUIRED`.
- `privacy` — another user's hidden section is `403` `PRIVATE` or `FRIENDS_ONLY`.
- `emitsToken` — the response contains access and refresh tokens. Do not log it.

## Status codes

| Code | Meaning |
|---|---|
| 200 | Success. Many writes are 200, not 201. Read the body: an "empty success" can still list `notFound` and `errored`. |
| 201 | Created, or a scrobble that crossed the watched threshold. |
| 302 | OAuth authorize redirect only. |
| 400 | Validation. Stable `error` codes include `INVALID_PARAMETER`, `INVALID_PAGE`, `INVALID_LIMIT`, `PAGE_TOO_DEEP`, `CURSOR_REQUIRED`, `CURSOR_NEEDS_COMPACT`, `JOURNAL_EXPIRED`. |
| 401 | Missing key, bad token, or bad client secret. `INVALID_TOKEN` was never issued. `TOKEN_REVOKED` was revoked. `MISSING_API_KEY` names the missing header. |
| 403 | `PRIVATE` or `FRIENDS_ONLY`. |
| 404 | Unknown id. |
| 409 | Conflict, such as a TMDB id that is both a movie and a show, or a device code already exchanged. |
| 410 | Device code expired. |
| 413 | `TOO_MANY_ITEMS` (over 5,000) or `PAYLOAD_TOO_LARGE` (over 1 MB). |
| 418 | User denied the device prompt. |
| 420 | `PLAN_LIMIT_REACHED`. Free quota. Body includes `message` and often `upgrade`. |
| 422 | `IDEMPOTENCY_KEY_REUSED`. |
| 423 | App key disabled. Support: https://wetrakr.com/support |
| 426 | `VIP_REQUIRED`. Same `upgrade` block as 420. |
| 429 | Per-minute limit, or daily quota once that test stops succeeding. Body `QUOTA_EXCEEDED` includes `upgrade_url` for free users. |

Some newer validation errors use `{ "success": false, "error": { "code": "...", "message": "..." } }`. Older ones use `error` as a string. Read `error` in both shapes.

`420` and `426` share:

```json
{
  "error": "VIP_REQUIRED",
  "message": "You need VIP to access this feature.",
  "upgrade": {
    "title": "Upgrade to VIP",
    "description": "Get VIP to unlock this feature.",
    "url": "https://wetrakr.com/vip"
  }
}
```

## Pagination

Default page size is 20. `limit` max is 100 on most lists, 1,000 on the journal, 5,000 on `compact=true`. `page` starts at 1. A bad page or limit is `400` `INVALID_PAGE` or `INVALID_LIMIT`. Discover rankings stop at 10,000 items: the next page is `400` `PAGE_TOO_DEEP`.

Response headers:

| Header | Meaning |
|---|---|
| `X-Pagination-Page` | Current page. |
| `X-Pagination-Limit` | Page size. |
| `X-Pagination-Page-Count` | Page count. |
| `X-Pagination-Item-Count` | Total items. On Discover rankings this stops at the 10,000-item depth. |
| `X-Pagination-Next` | Cursor for the next compact page. Send it back as `after`. |

`compact=true` accepts `true`, `1`, and `yes`. Compact rows are for sync: `type`, `id`, `ids`, and the fields that list adds. `extended` and `sort_by` are ignored. Walk them only with `after`. `page` with `compact=true` is `400` `CURSOR_REQUIRED`. `after` without `compact=true` is `400` `CURSOR_NEEDS_COMPACT`. Season and episode compact rows include `show_ids` (external ids of the show). Episode rows also carry `show_id`, `season_id`, `season_number`, and `number`.

Full pages are for drawing a screen. Compact pages are for knowing what changed.

## Dates

Timestamps are ISO 8601. Several `from_date` parameters also accept a Unix timestamp. Calendar and watched-day filters use `YYYY-MM-DD`. A `0` or a millisecond value written as seconds shows up as January 1970 and is flagged `watched_at_unknown` rather than treated as a real watch date.

## Filters

The same query names work on search, list items, tracking lists, a person's filmography, collection titles, and several Discover routes. A filter the route cannot apply is `400` `INVALID_PARAMETER` (changelog 1.0.7): that includes persons, lists, and comments, and popular / top rated / favorited when `filter_type` is missing.

Combine a list with commas: `genres_in=Drama,Comedy`. Ranges use `_min` / `_max` or `_from` / `_to`. A `0` budget or unknown runtime is excluded by a budget or runtime filter, not treated as zero.

Catalog filters (client id only):

| Parameter | Meaning |
|---|---|
| `title_search` | Narrow titles. On `GET /search` the text param is `q`, not `title_search`. |
| `rating_min`, `rating_max` | WeTrakr community score, 0 to 10. |
| `votes_min`, `votes_max` | Vote count. `votes_min` no longer drops titles above 3 million votes. |
| `genres_in` | Genre names, as in `GET /genres`. |
| `keywords_in` | Keyword names, as in `GET /keywords`. |
| `release_from`, `release_to` | `YYYY-MM-DD`, in the `wetrakr-api-country` region. |
| `runtime_min`, `runtime_max` | Minutes. Movie runtime, or episode runtime for a show. |
| `status_in` | Production `status_code` list. See below. |
| `budget_min_m`, `budget_max_m` | Millions of dollars. |
| `age_rating` | `G`, `PG`, `PG-13`, `R`, `NC-17`, `NR`. `PG-13` also matches `TV-14`. `R` also matches `TV-MA`. Other countries' certifications are not indexed. |
| `origin_country_in` | Production countries. |
| `original_language_in`, `original_language_not` | ISO 639-1. |
| `spoken_language_in`, `spoken_language_not` | ISO 639-1. |
| `hide_upcoming` | Boolean. |
| `premiered_types`, `dvd_digital_types`, `is_premiered`, `on_dvd_digital` | Release-type filters for movies in the country header region. |

`status_code`: `0` Planned, `1` Rumored, `2` In Production, `3` Post Production, `4` Released (movies), `5` Pilot, `6` Returning Series, `7` Ended, `8` Canceled (shows). Show shortcuts in the docs: returning series `6`, in production `2`, planned `0`, canceled `8`, ended `7`.

Personal filters need a user token:

| Parameter | Meaning |
|---|---|
| `user_rating_min`, `user_rating_max` | The caller's own rating, 0 to 10. |

Advanced filters are VIP. Without VIP: `426` `VIP_REQUIRED`.

| Parameter | Meaning |
|---|---|
| `tracking_in`, `tracking_not` | `watched`, `watching`, `playing`, `planning`, `paused`, `dropped`. |
| `streaming_in`, `streaming_not` | Provider ids. |
| `streaming_country` | Country for the provider list. |
| `actions_in`, `actions_not` | Other actions such as favorite or comment. |

Sort: `sort_by` plus `sort_dir` `asc` or `desc`. Unknown sort values on list items fall back to the owner's manual order. Rating sort can take `rating_source=imdb` or `rating_source=tmdb`.

## extended and append

`append` on a title detail embeds sub-resources: `credits`, `images`, `videos`, `rating`. Example: `GET /movies/126?append=credits,rating`. `rating` arrives as `rating_distribution` and does not replace the `rating` number (breaking change in 1.0.3).

`extended` is additive, not a ladder. `?extended=movie_level_1,movie_level_2` asks for both sets. You can mix types: `?extended=show_level_1,season_level_1`.

| Set | Adds |
|---|---|
| `movie_level_1` | `poster_path`, `backdrop_path`, `genres`, `runtime`, `status_code`, `rating`, `rating_votes`, `ratings`, `budget`, `revenue`, `people_watched`, `people_planning` |
| `movie_level_2` | `overview`, `tagline`, `homepage`, `popularity`, `production_companies`, `origin_country`, `adult` |
| `show_level_1` | `poster_path`, `backdrop_path`, `genres`, `tagline`, `status_code`, `rating`, `rating_votes`, `ratings`, `people_watched`, `people_planning` |
| `show_level_2` | `overview`, `homepage`, `created_by`, `number_of_seasons`, `number_of_episodes`, `production_companies`, `origin_country`, `adult` |
| `season_level_1` | `title`, `poster_path`, `rating_votes`, `people_watched`, `people_planning` |
| `season_level_2` | `overview` |
| `episode_level_1` | `title`, `still_path`, `runtime` |
| `episode_level_2` | `overview` |
| `user_personal_info` | `info.about` on a user |
| `user_account` | Nothing on other users. Kept so old clients do not fail. Preferences and timezone are not public. |
| `interactions` | The caller's state on a tracking-list row |

A full title is cheaper from its own detail route than from stacking every level onto a list. Compact mode ignores `extended`.

## Images

`poster_path`, `backdrop_path`, `still_path`, `profile_path`, `logo_path`, and `file_path` are TMDB paths.

`https://image.tmdb.org/t/p/{size}{path}`

Example: `https://image.tmdb.org/t/p/w500/qJ2tW6WMUDux911r6m7haRef0WH.jpg`

| Image | Sizes |
|---|---|
| Backdrop | `w300`, `w780`, `w1280`, `original` |
| Poster | `w92`, `w154`, `w185`, `w342`, `w500`, `w780`, `original` |
| Logo | `w45`, `w92`, `w154`, `w185`, `w300`, `w500`, `original` |
| Profile | `w45`, `w185`, `h632`, `original` |
| Still | `w92`, `w185`, `w300`, `original` |

Cache images. The path does not change. WeTrakr does not host these files. User avatars are the exception: `https://api.wetrakr.com/uploads/avatars/{file}`.

YouTube videos: `https://www.youtube.com/watch?v={key}` from `site` + `key`.

## Limits

Counted against the **user** when the request has an access token, otherwise against the **app key**.

| Quota | Free user | VIP user | App key, no token |
|---|---|---|---|
| Requests / day (test phase) | 1,000 | 50,000 | 1,000 |
| `GET` / minute | 200 | 1,000 | 200 |
| `POST`, `PUT`, `DELETE` / minute | 60 | 600 | 60 |

Daily quota is marked "Test phase". Requests over it still succeed for now. `X-Quota-Limit`, `X-Quota-Remaining`, and `X-Quota-Reset` (seconds until midnight UTC) show the position. Per-minute state is `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`. Develop keys start at the free column. Production keys have no daily quota and higher per-minute limits. Ask on Discord: https://discord.gg/XedDZYpFdh

Plan quotas that are **enforced** for free accounts only:

| Quota | Free | VIP |
|---|---|---|
| Lists owned | 5 | Not enforced |
| Items per list | 250 | Not enforced |
| Notes | 100 | Not enforced |
| Requests / day | 1,000 (test) | 50,000 (test) |

There is no tier above VIP, so VIP ceilings are not enforced. The `420` message and `GET /account/plan-usage` still quote the advertised numbers: 40 lists, 2,000 items per list, 1,000 notes. On VIP, `used` can be far past `limit`.

A free account over the cap does not delete lists. Extra lists are **locked**. A list over the item cap locks first. After that, the oldest imported lists lock before lists the user created by hand. `locked: true` and `locked_reason` `items` or `lists`. Visitors do not see locked lists. Item reads return `420`. Search and Discover drop them from results and totals. Followed lists the caller cannot read (VIP gate) come back `200` with `locked: true` and `locked_reason: followed`.

Batch writes: every entry in `movies`, `shows`, `seasons`, `episodes`, `people`, and every nested episode counts toward 5,000. Body cap is 1 MB.

## Commercial use

Personal and non-commercial apps are free. A service making 200 USD a month or more needs a commercial license. Ask on Discord. Do not route around the rate limit. Data from the API may not be used to promote or index infringing copies.
