# catalog endpoints

Extracted from the WeTrakr docs app on 2026-10-01 (changelog 1.0.7). Each route is `https://api.wetrakr.com` plus the path. Send `wetrakr-api-version: 1`.

Search, discover, calendar, people, collections, companies, networks, and static lists. Movie, show, season, and episode routes are in `references/catalog.md` (they are generated and not repeated here).

Auth `oauth` requires `Authorization: Bearer`. `apiKey` requires `wetrakr-api-key`. `optionalOauth` works with the key; a token adds the caller's state. An empty auth on a direct object means the symbols line is the source.

## Index

- `GET /calendar/movies` — Movie calendar
- `GET /calendar/shows` — Episode calendar
- `GET /collections/{id}/comments` — Get collection comments
- `GET /collections/{id}/images` — Get collection images
- `GET /collections/{id}` — Get a collection
- `GET /companies/{id}/images` — Get company images
- `GET /companies/{id}` — Get a company
- `GET /companies` — Companies
- `GET /countries` — Countries
- `GET /discover/favorited` — Most favorited
- `GET /discover/last` — Latest additions
- `GET /discover/listed` — Most listed
- `GET /discover/planning` — Most planned
- `GET /discover/popular` — Popular
- `GET /discover/ranks/{board_id}` — Rank board
- `GET /discover/ranks` — Ranks showcase
- `GET /discover/recommended` — Recommended for the user
- `GET /discover/toprated` — Top rated
- `GET /discover/trending` — Trending
- `GET /discover/watched` — Most watched
- `GET /genres` — Genres
- `GET /keywords` — Keywords
- `GET /languages` — Languages
- `GET /network/{id}/images` — Get network images
- `GET /network/{id}` — Get a network
- `GET /networks` — Networks
- `GET /people/{id}/collaborators` — Get frequent collaborators
- `GET /people/{id}/comments` — Get comments
- `GET /people/{id}/credits-grouped` — Get credits grouped by job
- `GET /people/{id}/credits` — Get credits
- `GET /people/{id}/images` — Get images
- `GET /people/{id}/known-for` — Get known for
- `GET /people/{id}/lists` — Get lists containing the person
- `GET /people/{id}/movies` — Get movies
- `GET /people/{id}/popularity-history` — Get popularity history
- `GET /people/{id}/shows` — Get shows
- `GET /people/{id}/stats` — Get filmography stats
- `GET /people/{id}/trivia` — Get trivia
- `GET /people/{id}` — Get a person
- `GET /search/all` — Quick search across types
- `GET /search/counters` — Count hits per type
- `GET /search/trending` — Trending searches
- `GET /search` — Search by type
- `GET /timezones` — Timezones

## GET /calendar/movies

- Docs id: `calendar-movies`
- Auth: `optionalOauth`

Movies released on each day of the window, grouped by day. Dates follow the region: the region parameter, else the user's country, else wetrakr-api-country. Each entry carries the release type and certification for that region. With a token, each movie carries the user's tracking state and rating.

### Query

- `region` (string, optional) — ISO 3166-1 country whose release dates to use.
- `release_type` (string, optional) — Comma-separated release types to include: 1 premiere, 2 limited, 3 theatrical, 4 digital, 5 physical, 6 TV. Default all.
- `start` (date, required) — First day of the window, YYYY-MM-DD.
- `end` (date, required) — Last day of the window, YYYY-MM-DD, on or after start. The window spans at most 31 days, both ends included; a longer one answers 400 INVALID_PARAMETER.
- `mode` (string, optional) — discover (default): every movie releasing, most popular first, with no cut. following: only movies the signed-in user tracks, which needs a token.

### Responses

- **200** — Days keyed by date, each an array of movies, plus the total.

## GET /calendar/shows

- Docs id: `calendar-shows`
- Auth: `optionalOauth`

Episodes airing on each day of the window, grouped by day, each with its show, network, local air time and the airing instant in UTC. Pass tz so episodes that cross midnight in the user's timezone land on the right day. With a token, mode=following limits it to shows the user tracks. The show block carries its external ids; status_code is the production status, with its meaning in the Media objects table of the Model page.

### Query

- `sort` (string, optional) — Order within a day: popularity (default in discover) or air_time (default in following).
- `tz` (string, optional) — IANA timezone of the user, for example Europe/Madrid. Days are bucketed in that zone. Without it days are whole UTC days; a timezone that does not exist answers 400 INVALID_TZ.
- `start` (date, required) — First day of the window, YYYY-MM-DD.
- `end` (date, required) — Last day of the window, YYYY-MM-DD, on or after start. The window spans at most 31 days, both ends included; a longer one answers 400 INVALID_PARAMETER.
- `mode` (string, optional) — discover (default): the 300 most popular shows releasing each day, most popular first; left_out in the answer counts the episodes of the shows past that cut. following: only titles the signed-in user tracks, which needs a token, with no cut.

### Responses

- **200** — Days keyed by date, each an array of episodes, plus the total and the sort applied.

## GET /collections/{id}/comments

- Docs id: `collections-comments`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `paginated`

Community comments written about the collection, 20 per page.

### Path params

- `id` (integer, required) — WeTrakr collection id, as found in belongs_to_collection on a movie.

### Query

- `page` (integer, optional) — Page number, starting at 1.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of comment objects with X-Pagination-* headers.

## GET /collections/{id}/images

- Docs id: `collections-images`
- Auth: `apiKey`
- Symbols: `apiKey`

Artwork of the collection grouped as backdrops, posters and others, as TMDB image paths. A collection without images gets empty groups.

### Path params

- `id` (integer, required) — WeTrakr collection id, as found in belongs_to_collection on a movie.

### Responses

- **200** — Image groups, empty when none.

## GET /collections/{id}

- Docs id: `collections-details`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

A franchise collection with its titles grouped by type: parts.movies and parts.shows, each a compact media object.

### Path params

- `id` (integer, required) — WeTrakr collection id, as found in belongs_to_collection on a movie.

### Query

- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — The collection.
- **404** — No collection with that id.

## GET /companies/{id}/images

- Docs id: `companies-images`
- Auth: `apiKey`
- Symbols: `apiKey`

Logos of the company grouped as logos and others, as TMDB image paths. A company without images gets empty groups.

### Path params

- `id` (integer, required) — WeTrakr company id, as found in production_companies on a title.

### Responses

- **200** — Image groups, empty when none.

## GET /companies/{id}

- Docs id: `companies-details`
- Auth: `apiKey`
- Symbols: `apiKey`

A production company: name, logo, country and external ids.

### Path params

- `id` (integer, required) — WeTrakr company id, as found in production_companies on a title.

### Responses

- **200** — The company.
- **404** — No company with that id.

## GET /companies

- Docs id: `static-companies`
- Auth: `apiKey`
- Symbols: `apiKey`, `paginated`

Production companies, with logo and country.

### Query

- `page` (integer, optional) — Page number, starting at 1.

## GET /countries

- Docs id: `static-countries`
- Auth: `apiKey`
- Symbols: `apiKey`, `paginated`

ISO 3166-1 countries with their English and native name, as used by origin_country, release dates and wetrakr-api-country. 251 entries.

### Query

- `page` (integer, optional) — Page number, starting at 1.

## GET /discover/favorited

- Docs id: `discover-favorited`
- Auth: `optionalOauth`

Titles and people added to favorites most often in the last days.

### Query

- `days` (integer, optional) — Window in days for the activity being counted, from 1 to 30. Default 7. Anything else answers 400 INVALID_PARAMETER.
- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects for one type, or an object keyed by plural type for several.

## GET /discover/last

- Docs id: `discover-last`
- Auth: `optionalOauth`

The newest entries in the catalog or the community, newest first. Exactly one type per call, with optional genre, year and department filters. With filter_type=list, extended=preview adds preview to each list: up to five of its items with title, ids, poster_path and backdrop_path.

### Query

- `filter_type` (string, required) — Exactly one of movie, show, person, list or comment.
- `filter_genres` (string, optional) — Comma-separated genre names, movies and shows only.
- `filter_year` (integer, optional) — Release year, movies and shows only.
- `filter_department` (string, optional) — known_for_department, people only, for example Acting.
- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of objects of the requested type with X-Pagination-* headers.
- **400** — Zero or more than one type, or an unknown one.

## GET /discover/listed

- Docs id: `discover-listed`
- Auth: `optionalOauth`

Titles and people that appear in the most public lists, counted over the last days.

### Query

- `days` (integer, optional) — Window in days for the activity being counted, from 1 to 30. Default 7. Anything else answers 400 INVALID_PARAMETER.
- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects with X-Pagination-* headers.

## GET /discover/planning

- Docs id: `discover-planning`
- Auth: `optionalOauth`

Titles added to plan-to-watch lists most often in the last days. A good "what is coming" signal.

### Query

- `days` (integer, optional) — Window in days for the activity being counted, from 1 to 30. Default 7. Anything else answers 400 INVALID_PARAMETER.
- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects for one type, or an object keyed by plural type for several.

## GET /discover/popular

- Docs id: `discover-popular`
- Auth: `optionalOauth`

The most popular titles right now by overall popularity score. Ask for several types at once to fill a home page in one call. Movies and shows take the catalog filters (genres_in, keywords_in, release_from, release_to, rating_min, votes_min, runtime_min, origin_country_in, original_language_in and the rest listed in Filters in Conventions) to rank only the titles that match. With filter_type=list, extended=preview adds preview to each list: up to five of its items with title, ids, poster_path and backdrop_path.

### Query

- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Several types: an object keyed by plural type.
- **200** — One type: a page of media objects with X-Pagination-* headers.

## GET /discover/ranks/{board_id}

- Docs id: `discover-ranks-board`
- Auth: `optionalOauth`

The full ranking behind one board of the showcase, paginated.

### Path params

- `board_id` (string, required) — top-rated-movies, top-rated-shows, most-watched-movies, most-watched-shows, most-watched-people, most-rated-movies, most-rated-shows, most-planned-movies, most-planned-shows, best-movie or best-show.

### Query

- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 40.
- `source` (string, optional) — Rating source for the rated boards. Default we.
- `window` (string, optional) — all (default), year, month or week.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of ranked items with X-Pagination-* headers.
- **400** — Unknown board.

## GET /discover/ranks

- Docs id: `discover-ranks`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

The community leaderboards in one call: top rated, most watched, most rated and most planned movies and shows, plus most watched people. Each board carries its top items with rank, previous rank and movement. Recomputed daily.

### Query

- `limit` (integer, optional) — Items per board, 1 to 20. Default 10.
- `source` (string, optional) — Rating source for the rated boards: we (WeTrakr, default) or an external one such as imdb, tmdb, letterboxd.
- `window` (string, optional) — all (default), year, month or week.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — The showcase.

## GET /discover/recommended

- Docs id: `discover-recommended`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`

Personalized recommendations built from what the user has watched, rated and favorited. Titles the user already tracks are excluded.

### Query

- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects for one type, or an object keyed by plural type for several.

## GET /discover/toprated

- Docs id: `discover-toprated`
- Auth: `optionalOauth`

Highest rated titles by the WeTrakr community: every title rated 8.5 or better, ranked by a weighted rating that pulls titles with few votes toward the average, so a single 10 cannot top the chart. The set and X-Pagination-Item-Count stay the same on every page and with any limit. Movies and shows take the catalog filters (genres_in, keywords_in, release_from, release_to, rating_min, votes_min, runtime_min, origin_country_in, original_language_in and the rest listed in Filters in Conventions) to rank only the titles that match.

### Query

- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects for one type, or an object keyed by plural type for several.

## GET /discover/trending

- Docs id: `discover-trending`
- Auth: `optionalOauth`

Titles with the most activity across the community in the last days: plays, ratings, favorites and list adds combined. One type per call. With filter_type=list, extended=preview adds preview to each list: up to five of its items with title, ids, poster_path and backdrop_path.

### Query

- `days` (integer, optional) — Window in days for the activity being counted, from 1 to 30. Default 7. Anything else answers 400 INVALID_PARAMETER.
- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects. Paging state comes in the X-Pagination-* headers.

## GET /discover/watched

- Docs id: `discover-watched`
- Auth: `optionalOauth`

Titles watched by the most users in the last days.

### Query

- `days` (integer, optional) — Window in days for the activity being counted, from 1 to 30. Default 7. Anything else answers 400 INVALID_PARAMETER.
- `page` (integer, optional) — Page number, starting at 1. Only honored when filter_type names a single type.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of media objects with X-Pagination-* headers.

## GET /genres

- Docs id: `static-genres`
- Auth: `apiKey`
- Symbols: `apiKey`, `paginated`

Genre names as they appear in media objects and in genres_in filters. 27 entries, movie and TV genres together.

### Query

- `page` (integer, optional) — Page number, starting at 1.

## GET /keywords

- Docs id: `static-keywords`
- Auth: `apiKey`
- Symbols: `apiKey`, `paginated`

Keyword tags used by keywords_in filters and the similar titles algorithm.

### Query

- `page` (integer, optional) — Page number, starting at 1.

## GET /languages

- Docs id: `static-languages`
- Auth: `apiKey`
- Symbols: `apiKey`, `paginated`

ISO 639-1 languages, as used by original_language, spoken_languages and wetrakr-api-language. 187 entries.

### Query

- `page` (integer, optional) — Page number, starting at 1.

## GET /network/{id}/images

- Docs id: `networks-images`
- Auth: `apiKey`
- Symbols: `apiKey`

Logos of the network grouped as logos and others, as TMDB image paths. A network without images gets empty groups.

### Path params

- `id` (integer, required) — WeTrakr network id, as found in networks on a show.

### Responses

- **200** — Image groups, empty when none.

## GET /network/{id}

- Docs id: `networks-details`
- Auth: `apiKey`
- Symbols: `apiKey`

A TV network: name, logo, country and external ids.

### Path params

- `id` (integer, required) — WeTrakr network id, as found in networks on a show.

### Responses

- **200** — The network.
- **404** — No network with that id.

## GET /networks

- Docs id: `static-networks`
- Auth: `apiKey`
- Symbols: `apiKey`, `paginated`

TV networks, with logo and country.

### Query

- `page` (integer, optional) — Page number, starting at 1.

## GET /people/{id}/collaborators

- Docs id: `people-collaborators`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

People who worked with this person most often, with the number of shared titles.

### Query

- `limit` (integer, optional) — Number of people. Default 12, maximum 40.

### Responses

- **200** — Collaborators, most shared titles first.

## GET /people/{id}/comments

- Docs id: `people-comments`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `paginated`

Community comments written about the person, 20 per page.

### Query

- `sort` (string, optional) — top (default), new or old.
- `page` (integer, optional) — Page number, starting at 1.

### Responses

- **200** — A page of comment objects. Paging state comes in the X-Pagination-* headers.

## GET /people/{id}/credits-grouped

- Docs id: `people-credits-grouped`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

The filmography grouped by job (Actor, Producer, Director...), each group with its titles newest first and the year and character or job on each. Built for a filmography page with one tab per job.

### Query

- `media_types` (string, optional) — Comma-separated: movie, show. Default both.

### Responses

- **200** — Groups of titles.

## GET /people/{id}/credits

- Docs id: `people-credits`
- Auth: `apiKey`
- Symbols: `apiKey`, `paginated`

Every credit of the person as a flat list, one row per role: cast, crew and guest star, across movies, shows and episodes. Each row embeds the compact title it belongs to under movie, show or episode.

### Query

- `filter_credit_type` (string, optional) — cast or crew.
- `filter_media_type` (string, optional) — movie or show.
- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — A page of credit rows. Paging state comes in the X-Pagination-* headers.

## GET /people/{id}/images

- Docs id: `people-images`
- Auth: `apiKey`

Images of the person grouped as profiles and others, as TMDB image paths. A person without images gets empty groups, not an error.

### Responses

- **200** — Image groups.

## GET /people/{id}/known-for

- Docs id: `people-known-for`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

The handful of titles the person is best known for, scored by popularity, votes and billing.

### Query

- `limit` (integer, optional) — Number of titles, 1 to 12. Default 6.

### Responses

- **200** — Best known titles.

## GET /people/{id}/lists

- Docs id: `people-lists`
- Auth: `apiKey`

Public lists that include this person.

### Query

- `extended` (string, optional) — Field sets to add to each media object, see Extended and append.

### Responses

- **200** — Array of list objects.

## GET /people/{id}/movies

- Docs id: `people-movies`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `paginated`

Every movie the person worked on, one entry per title with all their jobs on it. Accepts the catalog filters of Search and title_search. With a token, each title carries the user's interactions.

## GET /people/{id}/popularity-history

- Docs id: `people-popularity`
- Auth: `apiKey`

Daily popularity score and rank of the person over the last N days.

### Query

- `days` (integer, optional) — Window length in days, 1 to 90 (a larger value gives 90; the response echoes the days applied). Default 30.

### Responses

- **200** — One point per day.
- **400** — The id is not a number.
- **404** — No person with that id.

## GET /people/{id}/shows

- Docs id: `people-shows`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `paginated`

Every show the person worked on, one entry per title with all their jobs on it. Same filters as people movies.

## GET /people/{id}/stats

- Docs id: `people-stats`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

Numbers about the filmography: title counts, average rating, active years and decade, and the most frequent genres. The same block comes embedded in the person details.

### Responses

- **200** — Stats.

## GET /people/{id}/trivia

- Docs id: `people-trivia`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

Short trivia facts about the person. With a token each item carries the user's reactions.

### Responses

- **200** — Trivia items, possibly empty.

## GET /people/{id}

- Docs id: `people-details`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

Full profile of a person: biography, dates, external ids, popularity, and a filmography_stats block with title counts, average rating, active years and top genres. With a user token it also says whether the user favorited or commented the person and returns their private note.

### Query

- `language` (string, optional) — Locale for the biography when a translation exists, for example es-ES. Defaults to wetrakr-api-language.

### Responses

- **200** — The person object.

## GET /search/all

- Docs id: `search-all`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`

The search-bar endpoint: one call, every content type. Hits come back grouped as media (movies and shows), people and community (lists, collections and users), each as a compact object with a relevance score. Results are cached for five minutes per query. It always searches every type and takes no type filter: to search one type, such as users to turn a username into a user id, use GET /search with filter_type.

### Query

- `q` (string, required) — Text to search.

### Responses

- **200** — Grouped hits with per-type totals.
- **400** — Missing or too short q. Error bodies keep their envelope.

## GET /search/counters

- Docs id: `search-counters`
- Auth: `apiKey`

How many hits a query has in each content type. Use it to label tabs or decide which filter_type to show first.

### Query

- `q` (string, required) — Text to search, at least 1 character.

### Responses

- **200** — Hit count per type. company is always 0 for now.
- **400** — Missing q.

## GET /search/trending

- Docs id: `search-trending`
- Auth: `apiKey`

What people are searching for right now, ranked by distinct searchers and then hits over a rolling window. A search only shows up once at least 3 different people have made it, and user-name searches are never included.

### Query

- `limit` (integer, optional) — Number of queries. Default 20, maximum 100.
- `window_days` (integer, optional) — Window length in days, 1 to 90. Default 30.
- `filter_type` (string, optional) — Restrict to one type: movie, show, person, list, collection. Omit for all.

### Responses

- **200** — Trending queries, most searched first.

## GET /search

- Docs id: `search-text`
- Auth: `optionalOauth`
- Symbols: `optionalOauth`, `paginated`

Full-text search within one content type, with paging, sorting and the shared catalog filters. An IMDb (tt...) or Wikidata (Q...) id as the query resolves that exact title instead of searching text, and a plain number also matches TMDB, TVDB and WeTrakr ids. With a user token you can also filter by the user's own tracking state and ratings, and results carry their interactions.

### Query

- `q` (string, required) — Text to search, at least 1 character. An IMDb (tt...) or Wikidata (Q...) id resolves that title instead of searching; a plain number is searched as text and also matched against TMDB, TVDB and WeTrakr ids.
- `filter_type` (string, required) — What to search: media (movies and shows), movie, show, person, list, collection or user.
- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `sort_by` (string, optional) — Relevance by default. Options: popularity, avg_rating, votes, release_date, runtime, title, budget, revenue, people_watched, people_planning, birth_date (people only); with a token also my_rating and my_last_watched.
- `sort_dir` (string, optional) — desc (default) or asc.
- `rating_source` (string, optional) — With sort_by=avg_rating, sort by a third-party score instead of the WeTrakr one: imdb or tmdb.

### Responses

- **200** — A page of hits. Their shape depends on filter_type; paging state comes in the X-Pagination-* headers.
- **400** — Missing q or an unknown filter_type.

## GET /timezones

- Docs id: `static-timezones`
- Auth: `apiKey`
- Symbols: `apiKey`, `paginated`

IANA timezones accepted by the calendar tz parameter and the user location, one entry per country that uses each zone: a zone shared by several countries (Europe/London for GB, JE, GG and IM) comes once for each, so every country has its zones. Sorted by country. The UTC offset is not stored: it moves with daylight saving, so work it out from the name with Intl.

### Query

- `page` (integer, optional) — Page number, starting at 1.
