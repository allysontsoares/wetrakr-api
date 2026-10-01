# Changelog

Public API changelog from https://api.wetrakr.com/#/changelog as of 2026-10-01. Newest first. When a behavior here conflicts with an endpoint page, the endpoint page plus the newest changelog entry wins.

## 1.0.7 — 2026-10-01

- **Breaking.** Discover answers 400 INVALID_PARAMETER for catalog filters it cannot apply (on persons, lists or comments, on last, ranks, box office and played, and on popular, top rated and favorited without a filter_type) instead of ignoring them.
- **Breaking.** A comment needs text unless it carries a reaction or an attachment: an empty or blank text answers 400 INVALID_PARAMETER, and so does a text over 10,000 characters.
- **New.** GET /sync/last_activities carries journal_visible_until: how far the journal can be read now. The stamps move at write time and journal entries show up 5 seconds later, so a stamp newer than this is a change to read on the next cycle, not one to store as seen.
- **New.** GET /movies/{id}/stats and /shows/{id}/stats carry comments, favorites and lists totals next to tracking_list.
- **New.** GET /seasons/{id}/episodes carries community_rating on each episode (the WeTrakr community score from 0 to 10 and its votes), so a chart of scores per episode takes one call per season.
- **New.** Notes embed their title with its ids, and seasons and episodes their show with its ids.
- **New.** Every comment list sends X-Pagination-Item-Count, including list comments, replies, trending and last.
- **New.** extended=preview works on /discover/trending, /popular and /last with filter_type=list, and on GET /lists/{id} and /sync/lists/{id}.
- **New.** GET /users/{id}/following and /followers take page and limit, with X-Pagination-* headers. Without them the list comes whole, as before.
- **New.** Journal rows of ratings and notes carry the current value: rating, or note { text, spoiler }.
- **New.** Compact season and episode rows carry show_ids, the external ids of their show.
- **New.** GET /users/{id}/comments/{target} and /sync/comments/{target} take is_long=true (reviews) or is_long=false (short comments).
- **New.** /discover/trending, /favorited, /watched, /planning and /listed take the catalog filters (genres_in, release_from, rating_min, runtime, languages, countries, age_rating, status_in, keywords_in...) for movies and shows, like popular and top rated. Paging describes the filtered set.
- **New.** POST /sync/tracking answers added.plays: every play it created, with its play_id, so a single play can be removed at once with DELETE /sync/tracklogs/{play_id}.
- **Improved.** Those caught-up shows enter and leave watched in the journal as watched added and removed (the row has no play_id), and move shows.last_tracking_watched_at and last_tracking_removed_at, including when the user turns the setting on or off, which moves every caught-up show at once.
- **Improved.** The sync guide says that status moves of one title less than a second apart are journaled as their net change, and the status_in filter lists what each production status code means.
- **Improved.** The scrobble stop page explains the pause below 80 percent, the play without a start and the duplicate window, and the list pages explain extended=preview.
- **Fix.** GET /sync/lists/{id}/item-ids returns each item's own id: seasons and episodes came back with their show's id. Every row now carries its list_item_id too, and season and episode rows show_id.
- **Fix.** Plays dated in January 1970 (a 0 or a timestamp in the wrong unit from an import) are marked watched_at_unknown, and imports no longer write them.
- **Fix.** tracked_at_unknown and use_release_date work on nested episodes and seasons of POST /sync/tracking, also when set on the season or the show. The play was dated now and the flag was lost.
- **Fix.** ids.letterboxd is always a string. A slug made only of digits (2012, 300, 1917) came back as a number.
- **Fix.** Marking an episode watched no longer moves seasons.last_tracking_removed_at (and seasons.all) when its season is not complete yet. An internal row with no status was written and cleaned up on every episode, and both counted as changes.
- **Fix.** With nested seasons, a status on the show item no longer widens the scope: remove and remove/all take out only the nested seasons and episodes (the documented one-episode example removed the whole show), and on add a show with status watched no longer marks every episode watched.
- **Fix.** A 401 tells a token we never issued (INVALID_TOKEN) from one that was revoked (TOKEN_REVOKED). Both said Token revoked.
- **Fix.** An empty or blank note, or one over 10,000 characters, comes back in errored instead of being saved.
- **Fix.** activity.watch_time on profiles is refreshed when it is missing or older than a day. Some profiles showed 0 minutes despite thousands of plays.
- **Fix.** Marking a whole show watched counts the episode plays it creates in added.watched.episodes.
- **Fix.** Plays of episodes without an air date count for the show and season: a season-scope remove/all removes them, and the first play moves the show to watching.
- **Fix.** Tracking statuses no longer stack: writing the status a title already has changes nothing (it comes back under existing), and one remove or status none clears that status entirely instead of revealing an older one.
- **Fix.** Nested seasons and episodes that cannot be resolved come back in notFound (and invalid ones in errored) with their parent: { show: { id }, number } for a season, { show: { id }, season, number } for an episode, on every write (ratings, favorites, notes, tracking). They echoed only their own fields, so a batch with several shows could not tell which one failed.

## 1.0.6 — 2026-09-29

- **Breaking.** List names take up to 100 characters, and a name that is not text answers 400 INVALID_PARAMETER instead of Name is required.
- **Breaking.** Discover rankings answer 400 INVALID_PARAMETER for a filter_type they don't support (planning with person answered 200 {}), and popular and toprated answer 400 for title_search, which cannot be fast there: use /search with q.
- **Breaking.** POST /oauth/token/refresh answers 401 INVALID_REFRESH_TOKEN for a refresh token that will never work again (unknown, expired, revoked or of a disabled user), as documented. It answered 400.
- **Breaking.** status none clears a title's status but never deletes plays: on a watched title it comes back in errored (use /sync/tracking/remove). It used to delete the latest play and answer {}, and when it clears a status the answer now says which.
- **Breaking.** Every tracking item needs a status, on add and on every remove: an episode takes its own, else its season's, else its show's. Nested seasons and episodes without any used to be added as watched now, and top-level seasons without status were removed. They come back in errored (status is required), once per item.
- **Improved.** The reorder page lists its error codes, and the 413 TOO_MANY_ITEMS message says to send a longer order in chunks, last-first.
- **Fix.** A check-in that was not marked watched when its runtime ended (the job that does it only looks for 30 minutes) is now picked up by the hourly job instead of staying in playing forever.
- **Fix.** list_updated_at only moves when the list changes. Our nightly popularity ranking moved it on every list, so it could not be used as a change marker.
- **Fix.** Lists locked by the free plan limit are left out of GET /discover/trending, /discover/last, /lists/trending and /lists/last, as they already were from popular and search.
- **Fix.** GET /discover/trending with filter_type=comment returns the comments with the most replies and likes in the window. It always came back empty. Comments on private lists and hidden comments are left out.
- **Fix.** allow_comments false now closes a list to comments from everyone but its owner and collaborators: a comment or reply from anyone else answers 403 COMMENTS_DISABLED. It was stored but not checked.
- **Fix.** Privacy: the user lists of each title (watched, watching and favorited on movies, shows, seasons and episodes) leave out users whose profile or that section is not public. They were listed anyway. Your own row is always there.
- **Fix.** A compact cursor with a date more than 10 minutes ahead answers 400 INVALID_CURSOR. Up to a day ahead was accepted.
- **Fix.** GET /sync/favorites/lists answers 400 like any unsupported target. It answered 500.
- **Fix.** Another user's profile follows section privacy in activity.stats and watch_time too, and never shows notesCount.
- **Fix.** A duplicate scrobble stop at 100 (the play is already logged) changes nothing: it no longer moves movies.last_tracking_watched_at.
- **Fix.** POST /sync/ratings rounds every rating to one decimal, like the per-title routes. It only did when the batch also had an invalid value.
- **Fix.** GET /discover/trending with filter_type=person returns the people with the most comments, favorites and list adds in the window. It always came back empty.
- **Fix.** GET /discover/toprated counts X-Pagination-Item-Count from the same set as its rows, so a full walk always matches it. Popular with several filters (age_rating with release_from) answers in about a second.
- **Fix.** keywords_in takes keyword names in any case, with spaces or hyphens: time travel, Time Travel and time-travel match the same keyword.
- **Fix.** Release-type filters (premiered_types, dvd_digital_types, is_premiered, on_dvd_digital) work on /discover/popular, /discover/toprated and /search, for movies in the wetrakr-api-country region. Discover ignored them and search returned the same titles whatever the type.
- **Fix.** votes_min on list items no longer drops titles above 3 million votes, and a budget filter leaves out titles with an unknown budget (0).
- **Fix.** sort_by values that are not a real sort (watched, random, unknown ones) fall back to the owner's manual order on list items. They reversed it.
- **Fix.** A missing list answers the same 404 on every list route ({ error: NOT_FOUND, code: 404, message: List not found }), and a private or friends-only one the same 403 (This list is private).
- **Fix.** GET /sync/lists/item/{target}/{item_id} answers 404 for a title that doesn't exist or has another type, like its POST twin.
- **Fix.** Pinning or unpinning a list item writes a lists updated row and moves lists.last_updated_at and list_updated_at.
- **Fix.** Unticking a list in the add-to-list picker writes a lists updated row to the journal. Re-including an item that is already there, or removing one that is not, changes and journals nothing.
- **Fix.** The show and season watched removed row of the journal is written whenever the title leaves the watched list, also when the finale play is removed or right after rewatch-restart. It was missing now and then.
- **Fix.** Removes that remove nothing no longer move last_activities, updated_at or from_date feeds, and remove/last answers notFound and errored like remove, with currentTracks only for the items that lost a play.
- **Fix.** GET /discover/ranks carries the real computed_at of the boards, which are recomputed every day. It kept saying 2026-09-08.
- **Fix.** GET /sync/lists/{id}/items answers 404 List not found for this user for another user's list, private or public, like the other /sync/lists/{id} routes. A private one answered 403.

## 1.0.5 — 2026-09-29

- **Breaking.** GET /calendar/shows and /calendar/movies take a window of up to 31 days between start and end. A longer one answers 400 INVALID_PARAMETER: a year of shows was 75 MB, took over a minute and slowed every other request. Ask for longer ranges one month at a time.
- **Breaking.** Another user's hidden sections answer 403 with error PRIVATE or FRIENDS_ONLY: tracking, ratings, favorites and comments (lists and counters), and followers and following. They answered 200 with an empty list or zero counters, so a hidden section looked empty. Episode ratings keep hidden: true.
- **New.** Logos & colors: the official WeTrakr logos as SVG downloads, the brand colors and how to credit WeTrakr in your app, under Getting started.
- **Fix.** POST /sync/tracking/remove without tracked_at removes the latest play, as documented. It removed the earliest one, so undoing the last play of a rewatch deleted the first.
- **Fix.** A watched show that loses all its plays through a season or episode removal leaves the watched list, and the journal records it.
- **Fix.** POST /sync/lists/item/{target}/{item_id} resolves movies and shows by type: an id that is not of that type answers 404 instead of storing a row of the other type.
- **Fix.** Friends feed: plays and shows being watched fill their own slots, and a show someone keeps watching counts once, so one followee's import no longer empties everyone's plays.
- **Fix.** GET /discover/toprated breaks ties by id, so walking the pages no longer repeats some titles and skips others.
- **Fix.** List likes, comments and popularity history follow the list's privacy like the list itself: 403 with error PRIVATE or FRIENDS_ONLY, and 404 for a list that does not exist. The owner can read the likes of their own private list, and lists hidden by the owner's settings carry error PRIVATE too.
- **Fix.** A season sent without status to POST /sync/tracking comes back in errored (status is required), like movies, shows and episodes. It used to mark the season watched at the time of the call.
- **Fix.** Idempotency-Key compares bodies as JSON: the same body with its keys in another order is the same write, not a 422.
- **Fix.** Removing a season reports it once: episodes with nothing to remove are no longer listed one by one in errored, and the season comes back in errored only when nothing of it was removed.
- **Fix.** The last Discover page stops at the 10,000-item depth, so a full walk returns exactly what X-Pagination-Item-Count says.
- **Fix.** A compact cursor that could not have come from us (all zeros, all f, a date in the future) answers 400 INVALID_CURSOR instead of restarting or ending the walk.
- **Fix.** age_rating takes a comma-separated list and any case on search as well as Discover. The filter uses US certifications and their TV equivalents; the docs no longer suggest other countries' codes.
- **Fix.** GET /discover/toprated rows carry poster_path and backdrop_path again without extended.
- **Fix.** Moving a list with /sync/lists/{id}/rank or order/up|down writes lists updated in the journal for every list that moved, and list item writes that change nothing (removing a title already removed, re-including one already there) no longer do.
- **Fix.** GET /discover/planning, /watched and /favorited count only the titles they serve, so the last page is no longer short or empty.
- **Fix.** POST /sync/tracking/remove, /remove/last and /remove/all answer errored (status is required) for an item without status, instead of removing nothing silently.
- **Fix.** Your own profile, read with your token, counts all your lists, private ones included.

## 1.0.4 — 2026-09-29

- **Breaking.** days on Discover takes 1 to 30. A larger window (or a value that is not a whole number) answers 400 INVALID_PARAMETER: one call with days=365 took over a minute and slowed every other request.
- **Breaking.** A private or friends-only list or profile section answers 403 with error PRIVATE or FRIENDS_ONLY everywhere. Some routes (list items, counters and likes, another user's ratings or favorites) answered 400.
- **Improved.** Compact pages (compact=true) go up to 5,000 rows. Apps that sync whole libraries can ask on Discord for pages of up to 500 on the regular lists, for their first sync, once we have checked their sync flow: see Pagination.
- **Improved.** The same Idempotency-Key with a different body answers 422 IDEMPOTENCY_KEY_REUSED instead of replaying the first answer. 409 and 429 answers are no longer kept.
- **Improved.** POST /sync/lists/{id}/items/reorder errors carry a code: INVALID_PARAMETER, INVALID_ITEM_ID, DUPLICATE_ITEM, ITEM_NOT_IN_LIST and TOO_MANY_ITEMS.
- **Improved.** Scrobble start, pause and stop are applied as you send them. The 5% threshold, the watched lock and the abandoned-session rule are gone: a start at 0% or a pause on a watched title now opens a session (a rewatch) instead of answering ignored.
- **Improved.** The token refresh returns the new refresh token as refresh_token, the standard OAuth name used by the initial exchange, and keeps new_refresh_token with the same value.
- **Fix.** GET /users/{id} and its sub-routes answer 400 INVALID_PARAMETER when id is not a WeTrakr user id. With a token they used to answer the signed-in user's own profile and library, and /users/{id}/lists a 500.
- **Fix.** Filtered Discover pages answer in about a second: genres_in no longer takes 10 s, and a filter that matches nothing (an unknown genre, rating_min above 10, a release window in the future) answers an empty page at once instead of after 30-90 s or a 502. genres_in is matched in any case.
- **Fix.** GET /discover/toprated always serves the ranked set (titles rated 8.5 or better) on every page, with any limit and any filter. Deep pages and some filtered pages used to switch to every rated title, so the count changed mid-walk and titles rated 1.1 showed up.
- **Fix.** GET /account/stats/watched-time updates right after deleting a play, removing episode plays or moving a play to another date. Until now only new plays and whole titles leaving the watched list refreshed it, so the total could lag up to a day.
- **Fix.** Undoing a finished show (removing the finale play, unwatching a season) takes it out of the watched list, as the journal says. After rewatch-restart the show stays in watched, and the journal no longer says it left.
- **Fix.** Cancelling an episode session on a show in planning, dropped or paused leaves the show where it was. A show that was dropped and then paused used to go back to dropped.
- **Fix.** Removed person ratings come back with status removed and removed_at from the full from_date feeds, and no longer carry user_rating on /users/{id}/ratings.
- **Fix.** GET /calendar/shows in discover mode keeps the 300 most popular shows of each day, so a day and a week show the same episodes for that day. left_out counts the episodes past that cut.
- **Fix.** The friends feed is a fixed window of the 300 newest events: total and X-Pagination-* are the same on every page, and a busy followee no longer pushes out the recent events of others. Rating events use rated_at, undated imported plays and undone follows are left out, and types_in and user_id are validated (400 INVALID_PARAMETER).
- **Fix.** POST /sync/tracking: an item without status comes back in errored (status is required) instead of writing a play only the journal saw. The same dated play twice in one body is caught as Already added!, and a play logged without a date no longer blocks a dated play in the same second.
- **Fix.** POST /sync/tracking/season/watch-all on a season already watched no longer re-dates it to the time of the call.
- **Fix.** Adding a favorite that is already there changes nothing: a why sent again used to move the favorite's dates and last_activities while answering Already added!.
- **Fix.** One order/up or order/down nudge writes one lists updated row in the journal, not three. Season rows in the journal carry media_id.
- **Fix.** Discover paging headers stop at the 10,000-item depth: X-Pagination-Item-Count and Page-Count describe what you can walk before PAGE_TOO_DEEP.
- **Fix.** Discover filters follow the Filters guide: age_rating takes a comma-separated list (PG-13,R), origin_country_in takes codes in any case, and a malformed release_from or release_to is dropped instead of answering 400.
- **Fix.** INVALID_PARAMETER messages name the parameter you sent (days, release_from, id) instead of an internal field such as created_at or media_id.
- **Fix.** A second DELETE /users/{id}/follow on a follow you already removed answers 404 Follow not found and no longer moves the account stamps. POST /users/{id}/follow answers 201 on a new follow and 200 on a repeat, now documented.
- **Fix.** GET /discover/toprated honours extended, like popular.
- **Fix.** The five-item previews of /movies/{id}/lists and /shows/{id}/lists carry the dates of movies (release_date) and shows (first_air_date) on every call. Only one of the two used to come back, depending on the call, and the preview no longer carries the per-country release dates.
- **Fix.** Starting or pausing an episode of a show you do not track no longer puts the show in watching. The show enters watching when an episode is marked watched (a stop past the threshold or a check-in), as before.
- **Fix.** Privacy: the list count on another user's profile only counts the lists you can see, another user's preferences and timezone are no longer returned (profile and extended=user_account), and GET /users/{id}/lists shows your own private lists when the token is yours.
- **Fix.** Items sent under movies or shows are now resolved by type, so a title sent in the wrong list comes back in notFound instead of being stored with the other type. Remember that id is the WeTrakr id: a TMDB id goes in ids: { tmdb: ... }, and the two id spaces overlap.
- **Fix.** User search (GET /search?filter_type=user) finds every user: accounts created before the search index existed were missing from it.
- **Fix.** POST /sync/tracking/remove and /remove/all no longer answer an empty {} when nothing was removed: ids that do not exist come back in notFound, and items with nothing to remove in errored (Nothing to remove). They also take seasons and the nested show > seasons > episodes form, like add.

## 1.0.3 — 2026-09-27

- **Breaking.** Compact watch history rows (compact=true, released in 1.0.2) now use the same shape as every compact list: type, id and ids of the title, plus play_id and watched_at. movie_id and episode_id are gone.
- **Breaking.** append=rating on GET /movies/{id} and /shows/{id} embeds the distribution under rating_distribution. It used to replace the rating number with an object.
- **New.** Compact pages are walked with a cursor: send the X-Pagination-Next value back as after. Nothing shifts if the user writes during the sync, and deep pages cost the same. page is not accepted with compact=true (400 CURSOR_REQUIRED), including on the watch history.
- **New.** compact=true on tracking lists, ratings and favorites (yours and other users'): small rows (type, id, ids and what the list adds), 100 by default and up to 1,000 per page, for syncing. See Compact pages and cursors.
- **New.** GET /calendar/shows carries the external ids of each show in its show block.
- **New.** Writes take an Idempotency-Key header: repeating a write with the same key returns the answer of the first one instead of running it again, so a retry after a timeout never duplicates plays or lists. See Retries and idempotency.
- **New.** GET /account/friends/feed: what the people the user follows have been doing (plays, ratings, favorites, comments, lists, likes, follows), newest first, with each author's privacy settings applied.
- **New.** POST /sync/lists/{id}/items/reorder reorders many items of a list in one call. Moving an item with /items/{item_id}/rank now also shows up in the journal as lists updated.
- **New.** GET /discover/popular and /discover/toprated take the catalog filters (genres_in, release_from, votes_min, origin_country_in and the rest) for movies and shows, and rank only the titles that match.
- **Improved.** A request without the wetrakr-api-key header answers 401 with error MISSING_API_KEY and says what is missing, instead of the same No authorized! as a bad token.
- **Improved.** compact accepts 1 and yes as well as true. /movies/{id}/lists, /shows/{id}/lists and /seasons/{id}/lists honour limit (up to 100). The Join now link on the sign-in page keeps the PKCE challenge.
- **Improved.** Discover rankings go 10,000 items deep: a page past that answers 400 PAGE_TOO_DEEP at once, instead of taking seconds to return titles from the far end.
- **Fix.** GET /timezones lists every country that uses each zone (Europe/London for GB, JE, GG and IM), so no country is left without zones, and takes country to list the zones of one country.
- **Fix.** after without compact=true answers 400 CURSOR_NEEDS_COMPACT. Full pages ignored it and kept returning the same page.
- **Fix.** GET /sync/ratings/all, /sync/favorites/all and /sync/notes/all with from_date mark removed entries with status removed and removed_at, like the single-target lists. On /users/{id}/ratings/all each row also carries user_rating.
- **Fix.** GET /calendar/shows no longer returns episodes without their show block (shows left out as adult content, for example), and applies the default mode, discover, when mode is not sent.
- **Fix.** A second remove of a favorite, rating or note that is already removed no longer moves its removed_at.
- **Fix.** GET /discover/listed with filter_type season or person answers in seconds instead of hanging, and episode rows are compact episodes with their show.
- **Fix.** Detail responses follow wetrakr-api-country on every call: the first country to ask for a title is no longer served to everyone else.
- **Fix.** An unknown tracking status (for example "bogus") comes back in errored instead of being stored as the title's status.
- **Fix.** Starting or pausing an episode no longer takes its show out of planning, dropped or paused, so cancelling the session leaves the show where it was. GET /scrobble/playing and DELETE /scrobble/playing pick the session that is playing over an older paused one.
- **Fix.** Journal pages are no longer short or empty when a deleted list had rows: those rows are left out before paging, so Item-Count matches what you get.
- **Fix.** The journal records a show or season entering or leaving its watched list (category watched, without play_id). Marking a season or show watched no longer moves last_tracking_removed_at.
- **Fix.** Removing a rating moves <type>.last_rated_at again, and the ratings section of GET /sync/last_activities comes back (it was being dropped from the response).
- **Fix.** Privacy: when a user sets their lists to private, /lists/{id}/items and /lists/{id}/counters answer 403 like the list itself, and /sync/lists/{id} and /sync/lists/{id}/items answer 404 for a list that is not yours.
- **Fix.** After moving a play with POST /sync/tracking/update, the watched list shows the new date of the latest play instead of the old one.
- **Fix.** GET /calendar/shows without tz covers whole UTC days, and an unknown tz answers 400 INVALID_TZ instead of being ignored.
- **Fix.** GET /media/external with type=movie or type=show answers 404 when no title of that type has the id, instead of returning a person.
- **Fix.** Deleted notes come back from from_date as removed, without their text, and their text is no longer kept.
- **Fix.** extended=user_personal_info answers again and returns only the public bio (info.about). It never includes email or other account data.
- **Fix.** Requests with an invalid value no longer hang until the connection is cut: an id that is not a number, a malformed date in a body, an invalid enum and the like answer 400 INVALID_PARAMETER at once. An invalid from_date on ratings, favorites and lists answers 400 INVALID_FROM_DATE like the other sync feeds.
- **Fix.** Nested seasons and episodes in POST /sync/tracking that do not exist come back under notFound, with their show and numbers, instead of an empty success.
- **Fix.** Un-ignoring episodes in a batch through POST /sync/tracking/remove with status ignored works for every episode. Ignoring an episode that is already ignored writes nothing, so repeating ignore-all or ignore-previous changes nothing.
- **Fix.** A play sent again with the same date and time (to the second) comes back in errored as Already added! instead of being logged twice. POST /sync/tracking/season/watch-all skips episodes already watched, so calling it twice changes nothing, and the same title twice in one POST /sync/ratings body keeps the last value.

## 1.0.2 — 2026-09-27

- **Breaking.** No list comes back whole any more: without limit every paged endpoint returns 20 items, including target=all tracking lists, ratings and favorites. limit is capped at 100 (1,000 on the journal).
- **Breaking.** GET /oauth/authorize requires code_challenge (S256) and state for every app, and POST /oauth/token refuses a code issued without PKCE. The Google and Apple sign-in links now carry the challenge too.
- **Breaking.** POST /sync/tracking/update only moves the play that matches: tracked_at to the second, or the new play_id. When none matches, the item comes back in errored instead of moving the latest play. The response now counts updated plays.
- **New.** DELETE /scrobble/playing cancels a playback session without logging a play, and puts the title back where it was.
- **New.** GET /sync/tracking/watched/history/{target} takes compact=true: one small row per play (ids, numbers and date) and up to 1000 per page, for the first sync of a big library.
- **New.** Ignored episodes have their own sync signal: episodes.last_tracking_ignored_at in last activities, and the ignored category in the journal (added when an episode is ignored, removed when it is un-ignored).
- **New.** GET /users/{id}/ratings/{target} carries the score that user gave under user_rating, and GET /users/{id}/shows/{show_id}/episode-ratings returns its map of episode scores again (it came back empty).
- **New.** GET /sync/comments/{target} takes include_replies=true to list the user's replies too (each with parent_id), and the counters add replies.
- **Improved.** POST /scrobble/start, /checkin, /pause and /stop tell you what happened: an event WeTrakr filters out answers with ignored true and a reason, and a title it cannot resolve answers 404. They used to answer success either way.
- **Improved.** GET /search/trending only lists searches made by at least 3 different people.
- **Improved.** Scrobble accepts WeTrakr ids (movie.id, show.id, episode.id) and echoes the title and episode it resolved. An unknown episode answers 404, and a body with both movie and show answers 400.
- **Improved.** GET /sync/journal entries carry entry_id (a stable tie-break for entries with the same action_at), season_number and number on episodes, and watched_at_unknown on plays without a known date.
- **Fix.** A journal entry could appear later with an action_at older than entries already served, and a sync that stored the newest one skipped it. action_at is now the moment the entry is written, and the journal only serves entries a few seconds old.
- **Fix.** Removing a rating, a favorite, a note or a list item shows up in GET /sync/journal as removed, not updated.
- **Fix.** A page or limit that is not a whole number of 1 or more answers 400 INVALID_PAGE or INVALID_LIMIT, instead of a wrong slice, the whole list or a request that never answers.
- **Fix.** GET /lists/trending and GET /comments/trending/{target} are ordered by likes (the last 30 days first), not by date like the last feeds.
- **Fix.** Ratings go from 0 to 10 with one decimal (more are rounded). A value outside that range lands in errored on POST /sync/ratings and answers 400 on the per-title routes, instead of being stored.
- **Fix.** Rating a title through POST or DELETE /{type}/{id}/rating now works like /sync/ratings: the right type is stored, only that rating is removed, rating again replaces it, and it shows up in /sync, the journal and last activities. The 201 body is { message, rating: { target, id, rating, rated_at } }, and an unknown id answers 404.
- **Fix.** POST /scrobble/checkin now completes on its own: once the runtime has passed, the title is marked watched.
- **Fix.** Starting or checking in on a title closes the session that was playing, instead of leaving both as playing.
- **Fix.** Responses to apps are no longer cached at the edge by URL alone, so a signed-in call never gets an anonymous copy, the language headers are honoured, and every call is checked against your key and quota.
- **Fix.** GET /sync/lists/{id} only serves lists you own or collaborate on (404 otherwise; read other lists with GET /lists/{id}), and in /users/{id}/lists permissions describe you, not the profile owner.
- **Fix.** User objects no longer carry the user's privacy settings, gender or detected timezone outside GET /account/settings.
- **Fix.** Privacy is respected across public reads: comments on private or friends-only lists no longer show in the list, the comment feeds or GET /comments/{id}; /users/{id}/following and /followers follow the user's privacy; profile counters hide the sections the user keeps hidden, and never show how many notes they have.
- **Fix.** GET /sync/journal now includes plays from scrobble, check-in and the website play button, which were missing. Show-level rows no longer come as plays, and deleting a list with titles gives only its removed entry.
- **Fix.** The under-5% rule of scrobble start and pause applies to episodes as it does to movies: below 5% with no session open, the event is ignored. Episodes that already had a tracking row skipped it.
- **Fix.** Adding a favorite that is already there answers Already added! in errored and no longer bumps the public favorites counter.
- **Fix.** POST /sync/tracking/remove/all on a watched show also removes the plays of its episodes, as documented, and takes its seasons out of watched.
- **Fix.** Seasons in POST /sync/tracking work: a season marked watched, by id, by ids.tmdb or nested under its show, logs a play for each aired episode, and nested episodes by number are logged too. Before, every form answered {} and did nothing.
- **Fix.** The add-to-list picker (POST /sync/lists/item/{target}/{item_id}) answers 400 when an entry has no included, instead of taking the item out of that list. total_included_before and total_included count all your lists, not only the ones sent.
- **Fix.** POST /sync/lists/{id}/items/{item_id}/order/up moves the item toward rank 1, like the list-level order/up (it went the other way). At either edge both answer 400 in English.
- **Fix.** The playing list and its counters only hold movies and episodes: a show on hold (paused) no longer shows up in /sync/tracking/playing/shows.
- **Fix.** genres_in no longer depends on case: drama and Drama give the same titles.
- **Fix.** GET /sync/tracking/{status}/total-time answers 400 for an unknown target or seasons, instead of zeros that read as nothing watched.
- **Fix.** hide_upcoming no longer breaks other filters: on /movies/{id}/similar it kept only popular titles instead of similar ones, and on collections and companies it cancelled title_search.
- **Fix.** GET /discover/listed with filter_type season, episode or person returns seasons, episodes and people. It returned the movies that happened to share their ids.

## 1.0.1 — 2026-09-26

- **Fix.** GET /movies/{id}/stats and /shows/{id}/stats were answering zero on all five tracking lists, always. The count is real now: Breaking Bad reports 196,393 watched instead of 0.
- **Breaking.** In those stats the block is called tracking_list, not watchlist: they are the tracking lists, the same ones behind /sync/tracking. The old name still ships alongside it for now and will be removed.
- **Fix.** A comment id that is not a number now answers 400. It used to leave the request hanging with no reply at all, which you could hit just by getting a URL wrong.
- **Fix.** GET /sync/comments/counters counted your replies while GET /sync/comments/{target} did not list them, so the number and the list disagreed. Both now count the same thing.
- **Improved.** GET /comments/trending/{target} and /comments/last/{target} take the plural target (movies, shows) like the rest of the API, not only the singular. The singular keeps working.
- **Improved.** The stats of a title no longer mix internal markers into the tracking counts, and they include waiting, which was missing.
- **Fix.** Some endpoints were answering with the raw stored document, which carried internal fields and unflattened ids. GET /network/{id}, /seasons/{id}/videos and /seasons/{id}/episodes/{number} now follow the same public shape as everything else.
- **Fix.** The static tables were empty. GET /countries, /genres, /languages and /timezones now answer with 251 countries, 27 genres, 187 languages and 312 timezones.
- **Fix.** Who watched a season or an episode took up to 92 seconds and timed out on most clients. It now answers in under a second.
- **Fix.** The top rated, most favorited and most planned feeds in Discover were coming back empty for every type. They return titles again.
- **Improved.** Timezones carry the country they belong to. They do not carry a UTC offset: it moves with daylight saving, so work it out from the IANA name.
- **Breaking.** Who watched, who is watching and who favorited a title now count that title only, not its children. A show used to report every play of its episodes: Breaking Bad answered 196,393 watched where it has 2,873, the same number the show object carries in people_watched. Season favorites no longer include the favorites of its episodes.
- **Improved.** The top rated feed answers in under a second instead of twelve, with the same ranking. It also carries what a card needs: type, ids, artwork paths and dates, which it was not returning.
- **Improved.** Its X-Pagination-Item-Count now describes the ranked set (titles rated 8.5 or better) instead of every title with a single vote.
- **Improved.** The latest additions feed answers in a fraction of a second instead of fifteen. Same order, same items: it was sorting a million and a half titles with no index to return twenty.
- **Improved.** The popular feeds answer in a fifth of a second instead of three and a half. The result was already fast: what took the time was counting the whole catalog to fill X-Pagination-Item-Count on every page.
- **Breaking.** The advanced filters (tracking_in/not, streaming_in/not, actions_in/not) are VIP, as on wetrakr.com. Without the token of a VIP user they answer 426 VIP_REQUIRED. streaming_in used to work with no user at all.
- **Fix.** The 426 for adding collaborators now carries the upgrade block, like every other refusal by plan.
- **New.** GET /sync/tracking/watched/history/{movies|episodes}: every play, rewatches included, newest first, with the id of each play. The watched list still gives each title once.
- **Improved.** Each entry of GET /sync/tracking/playing/{target} now carries a playback block (status, progress_percent, runtime_seconds, tracked_at), so an app can build a Continue watching row across devices.
- **Breaking.** Leftovers of the review to comment rename are gone: review_added_at and review_updated_at are comment_added_at and comment_updated_at, allow_reviews is allow_comments, privacy.reviews is privacy.comments, and interactions.counter no longer repeats comments as reviews.
- **Breaking.** interactions.user.rating carries only rating and rated_at. Internal references (a raw database id where an object was expected) and storage dates (created_at, updated_at) no longer reach third-party apps.
- **Fix.** GET /discover/favorited, /discover/planning and the other community rankings return the compact objects the extended level asks for, not the whole stored record. Season and episode lists name their owner instead of an internal id.
- **Improved.** GET /sync/last_activities records tracking changes when they happen: moving a title between lists (planning to watched) moves last_tracking_removed_at as well as the list it went to, deleting one play moves it even when other plays remain, and editing a play date moves last_tracking_watched_at to the edit time. Timestamps only move forward and no field disappears when a list empties. Shows gain last_tracking_waiting_at.
- **Fix.** Editing the date of a play with POST /sync/tracking/update could move the title back to the list it was in before (planning) when the new date was earlier than that entry. The title now keeps its current list.
- **New.** GET /sync/journal: every item that changed since from_date, oldest first and paginated. Plays (watched, with play_id), tracking lists, ratings, favorites, notes, likes, comments and lists. It keeps 30 days.
- **Improved.** Batch writes take up to 5,000 items per call (the docs said 50, which was never enforced) and a body of up to 1 MB. Over the limit the call answers 413 TOO_MANY_ITEMS or 413 PAYLOAD_TOO_LARGE.
- **Fix.** A request body over the size limit, or one that was not valid JSON, went through as an empty body: a batch write too big answered 200 with nothing saved. It now answers 413 PAYLOAD_TOO_LARGE or 400 INVALID_JSON.
- **Fix.** GET /sync/lists and GET /sync/lists/item/{target}/{item_id} answer 200 with an empty array when the user has no lists, not 404.
- **Fix.** With from_date, removed ratings, favorites, likes and notes come back with status removed and removed_at, and deleted lists with status removed. Before they came back looking like any other entry.
- **Fix.** Ratings, favorites, notes, likes, comments and list changes move their GET /sync/last_activities timestamps when they happen. lists.last_updated_at never appeared before.

## 1.0.0 — 2026-09-24

- **Breaking.** GET /search/id is no longer part of the public API. GET /media/external/{source}/{external_id} does the same job: it answers with the WeTrakr id and type, caches for 12 hours and tells you when a TMDB id is shared by a movie and a show.
- **Breaking.** GET /search/all answers with the data object directly. The success, data and meta envelope is gone: what was in data is now the body. Error bodies keep their envelope.
- **New.** First public version of the API, with 246 documented endpoints and these docs. From here on, anything that changes what your app sees is written down here.
- **New.** from_date on the tracking lists, likes, notes and comments, so a sync can ask only for what changed since the last one.
- **Improved.** Reviews are called comments everywhere: /comments routes, comment_added_at and comment_updated_at, and a comments key in the responses. The old /reviews routes and field names keep working for now.
- **Improved.** Crossing a free plan quota now answers 420 with PLAN_LIMIT_REACHED, a message you can show and where to upgrade, instead of a silent empty result.
- **Improved.** Access tokens last 7 days and refresh tokens 180, so an app that a user opens every few weeks stops asking them to sign in again.
- **Fix.** last_activities now moves when a title leaves a tracking list, not only when it enters one, so an incremental sync no longer misses removals.
