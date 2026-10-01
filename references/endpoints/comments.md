# comments endpoints

Extracted from the WeTrakr docs app on 2026-10-01 (changelog 1.0.7). Each route is `https://api.wetrakr.com` plus the path. Send `wetrakr-api-version: 1`.

Community comments and the signed-in user's comments. Rules: `references/social.md`.

Auth `oauth` requires `Authorization: Bearer`. `apiKey` requires `wetrakr-api-key`. `optionalOauth` works with the key; a token adds the caller's state. An empty auth on a direct object means the symbols line is the source.

## Index

- `DELETE /comments/{id}/hide` — Unhide a comment
- `POST /comments/{id}/hide` — Hide a comment
- `DELETE /comments/{id}/like` — Remove a like
- `POST /comments/{id}/like` — Like a comment
- `GET /comments/{id}/likes` — Users who liked it
- `GET /comments/{id}/replies` — Get replies
- `POST /comments/{id}/replies` — Reply to a comment
- `GET /comments/{id}/translate` — Translate a comment
- `GET /comments/{id}` — Get a comment
- `PATCH /comments/{id}` — Edit a comment
- `GET /comments/last/{target}` — Latest comments
- `GET /comments/trending/{target}` — Trending comments
- `DELETE /sync/comments/{comment_id}` — Delete a comment
- `GET /sync/comments/{target}` — Get your comments
- `GET /sync/comments/counters` — Count your comments
- `POST /sync/comments/remove` — Remove comments by item
- `POST /sync/comments` — Write a comment

## DELETE /comments/{id}/hide

- Docs id: `comments-unhide`
- Auth: `optionalOauth`

Show the comment again for the signed-in user.

### Responses

- **200** — Unhidden.

## POST /comments/{id}/hide

- Docs id: `comments-hide`
- Auth: `optionalOauth`

Hide the comment for the signed-in user only. Hidden comments still come back in lists, marked hidden: true, so your UI can collapse them.

### Responses

- **200** — Hidden.

## DELETE /comments/{id}/like

- Docs id: `comments-unlike`
- Auth: `optionalOauth`

Take the like back.

### Responses

- **200** — Like removed.

## POST /comments/{id}/like

- Docs id: `comments-like`
- Auth: `optionalOauth`

Like the comment. Likes drive the trending feed.

### Responses

- **200** — Liked.

## GET /comments/{id}/likes

- Docs id: `comments-likes`
- Auth: `optionalOauth`

Users who liked the comment, most recent first.

### Responses

- **200** — Array of likes, each with a compact user and the date.

## GET /comments/{id}/replies

- Docs id: `comments-replies`
- Auth: `optionalOauth`

Replies to a comment, oldest first. Replies cannot have replies of their own: threads are one level deep.

### Responses

- **200** — Array of reply objects.

## POST /comments/{id}/replies

- Docs id: `comments-reply`
- Auth: `optionalOauth`

Post a reply on behalf of the signed-in user. Replying to a reply is not allowed.

### Body

- `text` (string, required) — The reply.
- `spoiler` (boolean, optional) — Flag it when it reveals plot.

### Responses

- **201** — The new reply.
- **400** — The target is itself a reply.
- **403** — The user is banned from commenting.
- **403** — The list has comments turned off (allow_comments false) and the user is not its owner or a collaborator.
- **404** — No comment with that id.

## GET /comments/{id}/translate

- Docs id: `comments-translate`
- Auth: `optionalOauth`
- Symbols: `oauth`

The comment text translated to a language. Translations are cached; the original comes back untouched when it is already in that language.

### Query

- `lang` (string, required) — Target language, ISO 639-1.

### Responses

- **200** — Translated text and where it came from: original, cache or the translation service.
- **400** — Missing or invalid lang.
- **503** — Translation service unavailable or quota exhausted.

## GET /comments/{id}

- Docs id: `comments-get`
- Auth: `optionalOauth`

One comment with its author, the title it is about, like and reply counts. With a token it also says whether the user liked or hid it.

### Responses

- **200** — The comment object.

## PATCH /comments/{id}

- Docs id: `comments-edit`
- Auth: `optionalOauth`

Change the text or the spoiler flag of one of the user's comments. Comments removed by moderation cannot be edited.

### Body

- `text` (string, required) — New text.
- `spoiler` (boolean, optional) — New spoiler flag.

### Responses

- **200** — Updated fields.
- **400** — Missing or empty text.
- **403** — Not the author, banned, or removed by moderation.

## GET /comments/last/{target}

- Docs id: `comments-last`
- Auth: `apiKey`
- Symbols: `apiKey`

The most recent comments for one target type.

### Path params

- `target` (string, required) — movie, show, season, episode, person, or all for every type at once. Singular, unlike the plural target of the other endpoints.

### Query

- `limit` (integer, optional) — Number of comments, 1 to 20. Default 5.

### Responses

- **200** — Array of comment objects, newest first.

## GET /comments/trending/{target}

- Docs id: `comments-trending`
- Auth: `apiKey`
- Symbols: `apiKey`

The most liked recent comments for one target type.

### Path params

- `target` (string, required) — movie, show, season, episode, person, or all for every type at once. Singular, unlike the plural target of the other endpoints.

### Query

- `limit` (integer, optional) — Number of comments, 1 to 20. Default 5.

### Responses

- **200** — Array of comment objects, each embedding its title.

## DELETE /sync/comments/{comment_id}

- Docs id: `account-comment-delete`
- Auth: `oauth`

Delete one comment or reply by id. Deleting a comment removes its replies too.

### Path params

- `comment_id` (integer, required) — Id of one of the user's comments or replies.

### Responses

- **200** — Deleted.
- **403** — Not the author.

## GET /sync/comments/{target}

- Docs id: `account-comments-list`
- Auth: `oauth`
- Symbols: `oauth`, `paginated`

Comments written by the user, newest first, 20 per page. Replies are left out unless you ask for them with include_replies=true.

### Path params

- `target` (string, required) — movies, shows, seasons, episodes, people, lists or all.

### Query

- `include_replies` (boolean, optional) — true to include the user's replies too, each with parent_id (the comment it answers). Use it to back up or mirror everything the user wrote.
- `page` (integer, optional) — Page number, starting at 1.
- `limit` (integer, optional) — Items per page. Default 20, maximum 100.
- `from_date` (string, optional) — Only what changed since this date, for incremental sync. ISO 8601 or a Unix timestamp. See Incremental sync.

### Responses

- **200** — A page of comment objects. Paging state comes in the X-Pagination-* headers.

## GET /sync/comments/counters

- Docs id: `account-comments-counters`
- Auth: `oauth`

How many comments the user wrote per target. Replies are counted apart, in replies, and are not part of all.

### Responses

- **200** — Counters per target.

## POST /sync/comments/remove

- Docs id: `account-comments-remove`
- Auth: `oauth`

Delete the user's comments on the given items, many at once. Same batched body as ratings: movies, shows, seasons, episodes, people arrays by id or ids.

### Body

- `shows` (array, optional) — Shows.
- `seasons` (array, optional) — Seasons by season id.
- `episodes` (array, optional) — Episodes by episode id.
- `people` (array, optional) — People by person id.

### Responses

- **200** — Counts per target.

## POST /sync/comments

- Docs id: `account-comments-add`
- Auth: `oauth`

Post a comment on a movie, show, season, episode, person or list. Texts of 200 words or more come back with is_long true and are shown as reviews. The language is detected automatically.

### Body

- `text` (string, required) — The comment, up to 10,000 characters. It can be empty only when the comment carries a reaction or an attachment; otherwise an empty or blank text answers 400 INVALID_PARAMETER.
- `spoiler` (boolean, optional) — Flag it when it reveals plot.
- `reaction` (string, optional) — Optional one-word reaction shown next to the text, for example like.

### Responses

- **201** — The new comment.
- **400** — No media object in the body.
- **403** — The user is banned from commenting.
- **403** — The list has comments turned off (allow_comments false) and the user is not its owner or a collaborator.
- **404** — The media object could not be resolved.
