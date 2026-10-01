[![skills.sh](https://skills.sh/b/allysontsoares/wetrakr-api)](https://skills.sh/allysontsoares/wetrakr-api/wetrakr-api)

# WeTrakr API

Agent skill for the [WeTrakr](https://wetrakr.com) public REST API at `https://api.wetrakr.com`. It teaches a coding agent the real routes, headers, auth, sync, scrobble, and plan limits before the agent writes a client or answers how a call behaves.

The skill follows the [Agent Skills](https://agentskills.io) format. `SKILL.md` is the router. The long reference stays in `references/` and is opened only for the area the task needs.

This is an independent summary of the public docs, captured on **2026-10-01** (changelog **1.0.7**, header `wetrakr-api-version: 1`). WeTrakr does not publish this package, and the docs site has no OpenAPI file. The pages at `/` and `/#/changelog` are one React app. If a live response disagrees with a reference, trust the response.

## Install

```bash
npx skills add allysontsoares/wetrakr-api
```

The CLI detects installed agents and copies the skill into the matching skills directory. Once it is installed, the agent loads it when the task matches the description in `SKILL.md`. You can also call it with `/wetrakr-api`.

Install for one agent, or for your user account instead of the current project:

```bash
npx skills add allysontsoares/wetrakr-api -a grok -y
npx skills add allysontsoares/wetrakr-api -g -y
```

`grok` is one target. The same `-a` flag accepts `claude-code`, `cursor`, `codex`, `github-copilot`, and the other agents listed by `npx skills`.

Preview the skill without installing it:

```bash
npx skills add allysontsoares/wetrakr-api --list
npx skills use allysontsoares/wetrakr-api
```

## Use when

- Integrating WeTrakr into an app, script, or media center
- Designing OAuth with PKCE, the device flow, refresh, or logout
- Syncing a library with `last_activities` and the journal
- Implementing tracking, ratings, favorites, notes, lists, or comments
- Scrobbling playback, or importing watch history
- Debugging a `401`, `403`, `420`, `422`, `426`, or `429` from `api.wetrakr.com`
- Checking whether a behavior changed in the public changelog

## What you can ask

```text
Add WeTrakr sign-in with PKCE and a rotating refresh token.
```

```text
Sync my watch history incrementally. Do not download the full lists.
```

```text
Scrobble this episode and mark it watched only when progress crosses 80%.
```

```text
Look up this TMDB id and tell me the WeTrakr id before you call the movie route.
```

## What the agent follows

These rules are in `SKILL.md`. The agent is told to open the matching reference before it invents a path, query name, or body field.

- Base URL is `https://api.wetrakr.com`. There is no `/api` or `/v1` prefix. The version travels in `wetrakr-api-version: 1`.
- Every call except the browser redirect sends `wetrakr-api-key`. Signed-in calls also send `Authorization: Bearer`.
- `id` is the WeTrakr id. A TMDB id goes in `ids.tmdb`. The two number spaces overlap. Resolve an external id with `GET /media/external/{imdb|tmdb|tvdb|letterboxd}/{id}`.
- `GET /movies/{id}` does not check type. A show id can come back as a show. Read `type`.
- Network detail is `GET /network/{id}`. The paged list is `GET /networks`.
- Sync order is `GET /sync/last_activities`, then `GET /sync/journal?from_date=`, then only the sections whose stamp moved. `400 JOURNAL_EXPIRED` means a full resync.
- Tracking statuses are `watching`, `waiting`, `watched`, `planning`, `dropped`, `paused`, and `playing`. `playing` comes from scrobble and is read-only. `waiting` is maintained by WeTrakr.
- Scrobble `stop` at progress `>= 80` marks the title watched (`201`). Below that it pauses (`200`).
- Batch writes accept at most 5,000 items and a 1 MB body.
- A hidden section on someone else's account is `403` with `PRIVATE` or `FRIENDS_ONLY`.
- Enforced free quotas are 5 lists, 250 items per list, and 100 notes. VIP-only actions return `426`. Do not retry `420` or `426`.
- Tokens, refresh tokens, and `client_secret` stay out of logs and commits. A wrong secret burns the authorization code.

## Reference map

| You are about to | Read |
|---|---|
| Log in, refresh, device flow, PKCE | `references/auth.md`, then `references/endpoints/auth.md` |
| Headers, errors, paging, filters, images, quotas | `references/conventions.md` |
| Sync, journal, tracking, ratings, favorites, notes, scrobble | `references/sync.md`, then `references/endpoints/sync.md` |
| Movies, shows, seasons, episodes, people, search, discover, calendar | `references/catalog.md`, then `references/endpoints/catalog.md` |
| Lists, comments, profiles, the signed-in account | `references/social.md`, then `references/endpoints/lists.md`, `comments.md`, and `account.md` |
| One route's query, body, and status codes | The matching file under `references/endpoints/` |
| Whether a behavior changed | `references/changelog.md` |

Search an endpoint file for the exact path, for example `` `GET /sync/journal` ``. The index at the top of each file is the route list.

## Layout

```text
SKILL.md                          router and the rules that override a guess
references/auth.md                PKCE, refresh, logout, device flow, external ids
references/conventions.md         headers, errors, paging, filters, images, quotas
references/sync.md                journal, tracking writes, scrobble
references/catalog.md             movie, show, season, and episode route matrix
references/social.md              lists, comments, profiles, account
references/changelog.md           public changelog 1.0.0 through 1.0.7
references/endpoints/             query, body, and status codes per route
```

Movie, show, season, and episode routes are generated in the docs app, so their matrix lives in `references/catalog.md`. The other routes are written one by one under `references/endpoints/`.

## Keeping the snapshot current

The references were read from the docs bundle on 2026-10-01. They were not verified with an authenticated call. When WeTrakr ships a newer changelog, update `references/changelog.md` and the page that owns the changed behavior, and leave the rest alone.

## License

[MIT](LICENSE)
