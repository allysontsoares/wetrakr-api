# Catalog

Docs start at https://api.wetrakr.com/#/movies . Search, discover, calendar, people, and static routes are listed with parameters in `references/endpoints/catalog.md`. Movie, show, season, and episode routes are generated in the docs app and are specified here.

Auth is `apiKey` unless noted. `optionalOauth` adds the caller's tracking, favorite, rating, note, and comment flag when a bearer token is sent. Rating writes are `oauth`.

## Lookup first

`GET /media/external/{source}/{external_id}` — see `references/auth.md`. On `GET /search`, a query that is an IMDb id (`tt…`), a Wikidata id (`Q…`), or a plain number resolves that title (the number also matches TMDB, TVDB, and WeTrakr ids) instead of running a text search.

`GET /movies/{id}` with a **show** id currently returns the show. It does not 404. Read `type`. The same is true in the other direction. Do not trust the path to enforce the type.

Text fields follow `wetrakr-api-language` when a translation exists. Release dates and certifications follow `wetrakr-api-country`.

`ids.letterboxd` is always a string, even when the slug is all digits (`2012`, `300`, `1917`).

## Movies and shows

Replace `{kind}` with `movies` or `shows`. `{id}` is the WeTrakr id.

| Method | Path | Auth | What it is |
|---|---|---|---|
| GET | `/{kind}/{id}` | apiKey, token optional for user state | Full metadata: titles, overview, dates, genres, artwork paths, external ids, community counters, third-party ratings. Query `append`: `credits`, `images`, `videos`, `rating`. `rating` embeds `rating_distribution` and leaves the `rating` number alone. |
| GET | `/{kind}/{id}/credits` | apiKey | Cast, guest stars, crew. Cast first, by billing. |
| GET | `/{kind}/{id}/images` | apiKey | `backdrops`, `logos`, `posters`, `others`. TMDB paths. See `references/conventions.md`. |
| GET | `/{kind}/{id}/videos` | apiKey | Trailers, teasers, clips, featurettes. `site` + `key`. |
| GET | `/{kind}/{id}/translations` | apiKey | Every localized title, overview, and tagline. |
| GET | `/{kind}/{id}/releases` | apiKey | Dates and certifications per country. Type: `1` premiere, `2` limited theatrical, `3` theatrical, `4` digital, `5` physical, `6` TV. |
| GET | `/{kind}/{id}/comments` | apiKey, token marks liked/hidden and hides blocked users | 20 per page. |
| GET | `/{kind}/{id}/lists` | apiKey | Public lists that include it, most popular first, 20 per page. Each list has a five-item preview. `limit` up to 100. Preview items include `release_date` on movies and `first_air_date` on shows. |
| GET | `/{kind}/{id}/rating` | apiKey | Community score and the vote histogram from 1 to 10. |
| POST | `/{kind}/{id}/rating` | oauth | Store the caller's rating. |
| DELETE | `/{kind}/{id}/rating` | oauth | Remove the caller's rating. |
| GET | `/{kind}/{id}/similar` | apiKey, token adds tracking state | TMDB recommendations first, then a keyword, genre, and franchise score. |
| GET | `/{kind}/{id}/stats` | apiKey | Community totals. As of 1.0.7 this includes comments, favorites, and lists next to `tracking_list`. |
| GET | `/{kind}/{id}/popularity-history` | apiKey | Daily popularity score and rank. Missing days repeat the last known value. Query window up to 90 days, default 30. |
| GET | `/{kind}/{id}/watching` | apiKey | Users currently watching it. 20 per page. |
| GET | `/{kind}/{id}/watched` | apiKey | Users who have it watched. |
| GET | `/{kind}/{id}/favorited` | apiKey | Users who favorited it. |
| GET | `/{kind}/{id}/tracking/history` | oauth | The caller's plays for this title, newest first, grouped by day. Each play has `_track_log_id` for `DELETE /sync/tracklogs/{id}`. |

Shows only:

| Method | Path | Notes |
|---|---|---|
| GET | `/shows/{id}/tracking/history/unified` | The show history plus every episode play, ordered by date. Episode rows put the episode id in `id`. |
| GET | `/shows/{id}/seasons` | All seasons. Specials are season `0`. Each season embeds a compact show. An airing season includes the next episode to air. A token adds tracking, favorite, rating, note, and comment. |
| GET | `/shows/{id}/seasons/{season_number}` | Compact season. `extended` adds title, poster, overview. `season_number` `0` is specials. |
| GET | `/shows/{id}/seasons/{season_number}/episodes/{episode_number}` | One episode by SxxExx. Use this when the scrobbler knows the code and not the WeTrakr episode id. `404` when that pair does not exist. |

Detail `404`: `{ "message": "Invalid id: Media not found", "code": 404 }`.

## Seasons and episodes

`{id}` here is the season id or the episode id, not the show id.

Both seasons and episodes:

| Method | Path | Notes |
|---|---|---|
| GET | `/{seasons\|episodes}/{id}` | Full record. |
| GET | `.../credits`, `.../images`, `.../videos`, `.../translations` | Same idea as movies. |
| GET | `.../comments`, `.../lists` | Community. |
| GET | `.../rating` | Distribution. |
| POST | `.../rating` | oauth. |
| DELETE | `.../rating` | oauth. |
| GET | `.../watched`, `.../favorited` | Users. |

Seasons only:

| Method | Path | Notes |
|---|---|---|
| GET | `/seasons/{id}/episodes` | Ordered by number. Each row has a compact show, `is_upcoming`, and `milestone`: `series_premiere`, `season_premiere`, `season_finale`, `series_finale`. `community_rating` is the WeTrakr score from 0 to 10 plus vote count, so a season chart is one call. Token adds tracking, favorite, rating, comment. `extended`: `episode_level_1`, `episode_level_2`, `show_level_1`, `show_level_2`. |
| GET | `/seasons/{id}/aired-status` | `{ total, aired, unreleased }`. Cheap "is this season finished" check. |
| GET | `/seasons/{id}/episodes/{episode_number}` | Full stored episode by number inside the season. `404` if that number is missing. |
| GET | `/seasons/{id}/watching` | Users watching it. Episodes do not have this route. |

Episodes only:

| Method | Path | Notes |
|---|---|---|
| GET | `/episodes/{id}/tracking/history` | Caller's plays, grouped by day, each with `_track_log_id`. oauth. |

Seasons do not have `/tracking/history`. Use the episode history or the show's unified history.

## People

Base `GET /people/{id}`. Token adds whether the caller favorited or commented, and their private note.

| Path | Notes |
|---|---|
| `/people/{id}/images` | `profiles` and `others`. Empty groups, not an error, when there are none. |
| `/people/{id}/credits` | Credit rows, paged. |
| `/people/{id}/movies` and `/shows` | One row per title with every job on it. Catalog filters plus `title_search`. `sort_by`: `popularity`, `avg_rating`, `votes`, `release_date`, `runtime`, `title`. Default release date, newest first. Token adds interactions. |
| `/people/{id}/credits-grouped` | Tabs by job, titles newest first, with year and character or job. |
| `/people/{id}/known-for` | Short list scored by popularity, votes, and billing. |
| `/people/{id}/collaborators` | People they worked with most, plus shared-title count. |
| `/people/{id}/stats` | Title counts, average rating, active years and decade, top genres. The same block is embedded on the person. |
| `/people/{id}/trivia` | Trivia items. |
| `/people/{id}/lists` | Public lists that include the person. |
| `/people/{id}/comments` | optionalOauth. |
| `/people/{id}/popularity-history` | Daily score and rank. |

Query `language` on the person overrides `wetrakr-api-language` for the biography.

## Search

| Path | Auth | Notes |
|---|---|---|
| `GET /search` | optionalOauth | One type. Required `q` (min 1 char) and `filter_type`: `media` (movies and shows), `movie`, `show`, `person`, `list`, `collection`, `user`. Paging, sort, catalog filters. Token unlocks personal filters. Default `limit` 20, max 100. |
| `GET /search/all` | optionalOauth | The search box. Grouped into media, people, and community (lists, collections, users). Compact objects with a relevance score. Cached 5 minutes per query. Always searches every group. |
| `GET /search/counters` | apiKey | Hit counts per type, for tabs. |
| `GET /search/trending` | oauth | Ranked by distinct searchers, then hits. A query appears only after **3** different people searched it. User-name searches are excluded. |

## Discover

`GET /discover/...`. Most are optionalOauth and paginated. Rankings are 10,000 items deep.

| Path | Notes |
|---|---|
| `/discover/trending` | Plays, ratings, favorites, and list adds over the last days. One type per call. `filter_type=list` plus `extended=preview` adds up to five items (title, ids, poster, backdrop). |
| `/discover/popular` | Popularity score. Several types in one call. Movies and shows take catalog filters. |
| `/discover/toprated` | Community score at least 8.5, weighted so a single 10 cannot lead. The set and `X-Pagination-Item-Count` stay stable across pages and limits. `extended` is honored. |
| `/discover/favorited` | Most favorited titles and people in the window. |
| `/discover/watched` | Watched by the most users in the window. |
| `/discover/planning` | Most added to planning in the window. |
| `/discover/listed` | Most often present in public lists. |
| `/discover/last` | Newest catalog or community entries. One type. Optional genre, year, department. List preview works the same as trending. |
| `/discover/recommended` | **oauth**. From what the user watched, rated, and favorited. Already-tracked titles are excluded. |
| `/discover/ranks` | Showcase boards. |
| `/discover/ranks/{board_id}` | One board, paginated. Unknown board errors. |

Catalog filters on trending, favorited, watched, planning, and listed apply to movies and shows. Using them where they cannot apply is `400` `INVALID_PARAMETER`.

## Calendar

Both optionalOauth. Window is at most **31 days** between `start` and `end`. Longer is `400` `INVALID_PARAMETER`.

| Path | Notes |
|---|---|
| `GET /calendar/movies` | Grouped by day. Region: `region` query, else the user's country, else `wetrakr-api-country`. Each entry has the release type and certification for that region. A token adds tracking. |
| `GET /calendar/shows` | Episodes grouped by day, with show, network, local air time, and UTC instant. Pass `tz` (IANA, from `GET /timezones`) so a midnight crossing lands on the right day. `mode=following` with a token limits to shows the user tracks. |

## Collections, companies, networks, static

| Path | Notes |
|---|---|
| `GET /collections/{id}` | Franchise. `parts.movies` and `parts.shows`, compact media. optionalOauth. |
| `GET /collections/{id}/comments` | 20 per page. apiKey. |
| `GET /collections/{id}/images` | `backdrops`, `posters`, `others`. Empty groups if none. |
| `GET /companies` | Paged production companies. |
| `GET /companies/{id}` | Name, logo, country, external ids. |
| `GET /companies/{id}/images` | `logos`, `others`. |
| `GET /networks` | Paged TV networks. |
| `GET /network/{id}` | Singular `network`. Name, logo, country, external ids. |
| `GET /network/{id}/images` | Logos. |
| `GET /countries` | 251 ISO 3166-1 rows. English and native name. |
| `GET /genres` | 27 names, movies and TV together. `ids.tmdb`. |
| `GET /languages` | 187 ISO 639-1 rows. |
| `GET /timezones` | IANA zones, one row per country that uses the zone, sorted by country. Offset is not stored. |
| `GET /keywords` | Tags used by `keywords_in` and by similar-titles. |

Static lists are paged (`page`). They are the legal values for the matching filters.
