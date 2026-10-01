# wetrakr-api

Agent skill for the [WeTrakr](https://wetrakr.com) public REST API at `https://api.wetrakr.com`.

This is an independent reference, written from the public docs on **2026-10-01** (changelog **1.0.7**, header `wetrakr-api-version: 1`). It is not published by WeTrakr. There is no official OpenAPI file. If a live response disagrees with the skill, trust the response.

## Install

```bash
npx skills add <owner>/wetrakr-api
```

The skill lives at the repository root (`SKILL.md` plus `references/`). Agents that follow the [Agent Skills](https://agentskills.io) layout can also read it from `.grok/skills/` after install.

## What the agent gets

`SKILL.md` is the router: base URL, headers, id spaces, sync order, scrobble, batch limits, and privacy errors. The detail sits in `references/`:

| File | Contents |
|---|---|
| `references/conventions.md` | Headers, errors, paging, filters, images, quotas |
| `references/auth.md` | PKCE, refresh, logout, device flow, external ids |
| `references/sync.md` | Journal, tracking, ratings, favorites, notes, scrobble |
| `references/catalog.md` | Movies, shows, seasons, episodes, search, discover, calendar |
| `references/social.md` | Lists, comments, profiles, account |
| `references/changelog.md` | Public changelog 1.0.0 through 1.0.7 |
| `references/endpoints/` | Query, body, and status codes per route |

## License

[MIT](LICENSE)
