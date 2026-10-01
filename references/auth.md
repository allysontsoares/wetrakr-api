# Authentication

Docs: https://api.wetrakr.com/#/authentication

Parameter tables and status codes: `references/endpoints/auth.md`.

Two ways to get a user token. Both end at the same access token and refresh token. Access tokens last **7 days**. Refresh tokens last **180 days**. Refresh rotates the refresh token. Store the new one. The previous one dies after a short grace (a refresh of a token rotated more than a minute ago is `401` and will not work again).

Public clients (mobile, desktop, browser, TV) prove the request with PKCE and **omit** `client_secret`. Send `client_secret` only from a server you control. A wrong secret is `401` and **burns** the authorization code. Start the flow again.

Create the app key in WeTrakr settings. The client id is `wetrakr-api-key` and `client_id`. A second product needs its own account and key. Usage for the key is on the app page in Settings.

## Authorization code + PKCE

1. Generate `code_verifier` (high-entropy string).
2. `code_challenge = BASE64URL(SHA-256(code_verifier))` with no padding. The challenge is 43 characters. `code_challenge_method` is `S256`. Anything else is `400`.
3. Generate `state` and keep it. Verify it on the redirect.
4. Send the user, in a browser, to `GET /oauth/authorize` with `client_id`, `redirect_uri`, `code_challenge`, `code_challenge_method=S256`, and `state`. This is HTML, not JSON. `redirect_uri` must be registered on the app.
5. WeTrakr redirects to `redirect_uri` with a one-time `code` and your `state`.
6. Immediately `POST /oauth/token` with JSON body `client_id`, `code`, `code_verifier`, and `client_secret` only for a confidential server. Codes are single-use and short-lived.
7. Persist `access_token` and `refresh_token`. Do not log them.

`GET /oauth/authorize` without PKCE is refused, and the token endpoint also refuses a code that was issued without PKCE.

## Refresh and logout

`POST /oauth/token/refresh` with `{ "refresh_token": "..." }`. Response is a new access token and a new refresh token. Replace both.

`POST /oauth/logout` with `Authorization: Bearer <access_token>` and body `{ "refresh_token": "..." }`.

A dead refresh token will not recover. Send the user through authorize again. Distinguish `INVALID_TOKEN` from `TOKEN_REVOKED` on `401`.

## Device flow

For a TV or any device with no browser.

1. `POST /oauth/device/code` with `{ "client_id": "..." }`. At most **5 codes every 15 minutes per IP** (`429`, wait 15 minutes). `423` means the app is suspended.
2. Show `user_code` and open or display `verification_url`. Keep `device_code`. Poll every `interval` seconds until `expires_in`.
3. `POST /oauth/device/token` with `client_id`, `device_code`, and `client_secret` only from a server.

Poll results:

| Status | What to do |
|---|---|
| 200 | Approved. Save both tokens. Stop. The code cannot be reused (`409` if you try). |
| 400 and `authorization_pending` | User has not answered. Wait `interval`, poll again. |
| 400 without that error | Your body is wrong (`device_code` or `client_id` missing). Stop. |
| 401 | Wrong `client_secret`. |
| 404 | Unknown code, or the code belongs to another app. |
| 410 | Expired. Start at device/code again. |
| 418 | User denied. Stop. |
| 429 | Polled faster than `interval`. Slow down, then continue. |

## External ids

`GET /media/external/{source}/{external_id}`

`source`: `imdb`, `tmdb`, `tvdb`, `letterboxd`. Letterboxd is a slug (`the-dark-knight`). IMDb ids starting with `nm` resolve to people. Optional query `type=movie|show|person` is required only when a TMDB id is used by both a movie and a show (`409` lists both matches).

Use the returned WeTrakr id on every other route. Do not put a TMDB id in the `id` field of a write body.
