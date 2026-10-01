---
name: wetrakr-api
description: >
  Reference for the WeTrakr public REST API at api.wetrakr.com: OAuth and PKCE,
  headers, sync and journal, scrobble, tracking, ratings, lists, comments,
  catalog, discover, and rate limits. Use when integrating, calling, debugging,
  or designing against WeTrakr, or when the user says WeTrakr API, api.wetrakr.com,
  wetrakr sync, scrobble, watch history import, or /wetrakr-api.
---

# WeTrakr API

Consult this skill before calling `https://api.wetrakr.com`, designing a client, or answering how a WeTrakr route behaves. Do not invent paths, query names, status values, or body fields. Open the reference that owns the area, then follow it.

Captured from the public docs app on **2026-10-01**, changelog **1.0.7**, header `wetrakr-api-version: 1`. There is no published OpenAPI file. The site at `/` and `/#/changelog` is one React app. If a live response contradicts a reference, trust the response, name the stale file, and do not silently "fix" the skill.

Paths below are relative to this skill directory.

## Which file to open

| You are about to | Read |
|---|---|
| Log in, refresh, device flow, PKCE | `references/auth.md`, then `references/endpoints/auth.md` |
| Headers, errors, paging, filters, images, quotas | `references/conventions.md` |
| Sync, journal, tracking, ratings, favorites, notes, scrobble | `references/sync.md`, then `references/endpoints/sync.md` |
| Movies, shows, seasons, episodes, people, search, discover, calendar | `references/catalog.md`, then `references/endpoints/catalog.md` |
| Lists, comments, profiles, the signed-in account | `references/social.md`, then `references/endpoints/lists.md`, `comments.md`, `account.md` |
| One route's query, body, and status codes | The matching file under `references/endpoints/` |
| Whether a behavior changed | `references/changelog.md` |

Search the endpoint file for the exact path, for example `` `GET /sync/journal` ``. The index at the top of each file is the route list.

## Rules that override a guess

1. Base URL is `https://api.wetrakr.com`. There is no `/api` or `/v1` prefix. The version is the header `wetrakr-api-version: 1`.
2. Send `wetrakr-api-key` (the app client id) on every call except the browser redirect. Send `Authorization: Bearer <access_token>` when auth is `oauth`, and whenever you want the caller's own state on `optionalOauth`.
3. `id` is the WeTrakr id. A TMDB id is `ids.tmdb`. The two number spaces overlap. Resolve with `GET /media/external/{source}/{external_id}` (`imdb`, `tmdb`, `tvdb`, `letterboxd`). IMDb `nm…` is a person. A TMDB id that is both a movie and a show returns `409` until you pass `type`.
4. `GET /movies/{id}` does not check type. A show id returns the show. Read `type` on the object.
5. Network detail is `GET /network/{id}` (singular). The paged list is `GET /networks`.
6. Sync in this order: `GET /sync/last_activities`, then `GET /sync/journal?from_date=`, then only the sections whose stamp moved. Do not poll full lists. Journal rows show up about 5 seconds after the stamp. Honor `journal_visible_until`. `400 JOURNAL_EXPIRED` means a full resync, then journal from the new mark.
7. Tracking statuses: `watching`, `waiting`, `watched`, `planning`, `dropped`, `paused`, `playing`. Allowed targets per status are in `references/sync.md`. One title sits in one list. `playing` is read-only and comes from scrobble. `waiting` is maintained by WeTrakr.
8. Scrobble `stop` at progress `>= 80` marks watched (`201`). Below that it pauses (`200`). Only one title plays at a time.
9. Batch writes accept at most **5,000** items and a **1 MB** body. Over that: `413` `TOO_MANY_ITEMS` or `PAYLOAD_TOO_LARGE`.
10. Another user's hidden section is `403` with `PRIVATE` or `FRIENDS_ONLY`, not an empty `200`.
11. Enforced free quotas: **5** lists, **250** items per list, **100** notes. VIP is not blocked by those quotas. The `420` text may quote 40 lists, 2,000 items, or 1,000 notes. Those figures are not what the API enforces. VIP-only actions return `426` `VIP_REQUIRED`. A locked list can still be `200` with `locked: true`. Reading its items is `420`.
12. Back off on `429`. Per-minute limits are enforced. The daily quota is in a test phase and a request over it may still succeed. Read `RateLimit-*` and `X-Quota-*`.
13. Poster and backdrop paths are TMDB: `https://image.tmdb.org/t/p/{size}{path}`. User avatars are `https://api.wetrakr.com/uploads/avatars/{file}`.
14. Do not log or commit `client_secret`, access tokens, or refresh tokens. Public apps use PKCE and omit `client_secret`. A wrong secret burns the authorization code.
15. Do not retry `420` or `426`. Show `message` and the `upgrade` block.

## When writing code

- Call only the routes the task needs. Match the project's HTTP client. Do not add a WeTrakr SDK wrapper for a single call.
- Paginate with the headers in `references/conventions.md`. Compact sync uses the `after` cursor from `X-Pagination-Next`. `page` together with `compact=true` is `400 CURSOR_REQUIRED`.
- After an implementation, name the routes and the reference file you followed.
- Do not send a request until a client id is available from the user or from project configuration you have actually read.
