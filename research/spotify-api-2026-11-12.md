# Spotify account linking before 250k MAU: what Fuse can actually do (2026-11-12)

All sources were fetched on **2026-11-12** unless marked otherwise. "Unverified" means I could not confirm it on an official page.

## Recommendation

**Building Spotify linking now is fine as an internal experiment, but don't make it a launch feature.** The 250k MAU bar isn't a general Web API gate. It's one of six requirements for **Extended Quota Mode**, and Extended Quota Mode is the only way to let more than 5 users link their Spotify accounts ([quota-modes](https://developer.spotify.com/documentation/web-api/concepts/quota-modes)). A new app stays in **Development Mode** until then. Development Mode has these limits:

- Up to **5 allowlisted users** per app.
- The app owner must have **Spotify Premium**.
- Spotify says it's for "non-commercial use by individual developers" and "should not be relied on as a foundation for building or scaling a business on Spotify" ([Feb 2026 blog](https://developer.spotify.com/blog/2026-02-06-update-on-developer-access-and-platform-security)).

There's no official workaround, startup tier or early-access program for Spotify. Scraping, unofficial clients and fake MAU all break Spotify's terms, so they aren't an option. Separately, Spotify's Developer Policy bans "building profiles of users" and "commercial product offers" ([policy III.12–13](https://developer.spotify.com/policy)). Get a legal read on those two clauses before you design group taste matching around Spotify data.

**What to do instead:**
- **Launch:** build a first-party "taste" step in onboarding (genres, artists, vibe), with no third-party dependency.
- **Optional:** add Apple Music (MusicKit) linking, which has no MAU gate that I could find.
- **Now:** keep a Spotify Dev Mode prototype for the team (≤5 users) so the plumbing is ready.
- **Later:** apply for Extended Quota once Fuse is a launched, registered business that is close to 250k MAU.

---

## 1. What the 250k MAU requirement is

| Item | Finding | Source |
|---|---|---|
| Program | **Web API Extended Quota Mode**, the "Quota Extension Request for New Potential Partners" | [quota-modes](https://developer.spotify.com/documentation/web-api/concepts/quota-modes) |
| Exact wording | "Implementation requirements: 1. Established Business Entity (legally registered business or organisation) 2. Operating an active, and Launched Service 3. Maintaining a minimum of active users (**at least 250k MAUs**) 4. Being available in key Spotify markets 5. Commercial Viability 6. Adherence to Terms" | same |
| Who can apply | "as of May 15th 2025, Spotify only accepts applications from organizations (not individuals)". You apply from a company email through the [partner form](https://docs.google.com/forms/d/1O87xdPP1zWUDyHnduwbEFpcjA57JOaefCgBShKjAqlo) | same |
| Why it exists | Since May 15, 2025, extended access has been "reserved for apps with established, scalable, and impactful use cases that help drive our platform strategy forward and promote artists and creator discovery". Spotify says "over 95%" of past applications failed basic security, privacy and licensing standards | [Apr 15 2025 blog](https://developer.spotify.com/blog/2025-04-15-updating-the-criteria-for-web-api-extended-access) |
| What it gates | **Using the app with more than 5 users**, plus higher rate limits. It doesn't gate the Web API itself, because Dev Mode works without it. "Extended quota mode … can be installed by an unlimited number of users and the allowlist in development mode no longer applies" | [quota-modes](https://developer.spotify.com/documentation/web-api/concepts/quota-modes) |
| After the form | If the partner form is approved, you submit the in-dashboard Quota Extension questionnaire, which has 4 steps. App review then checks the app against the Developer Policy, which "can take up to six weeks" | same |
| Guarantees | "Spotify makes no guarantees or promises on processing times, or that such extended access request will be approved" | [policy VII.3](https://developer.spotify.com/policy) |

- **Does Fuse qualify today?** No. Fuse is pre-launch and well below 250k MAU.
- **Do projected users count?** A volunteer moderator on Spotify's community forum suggested Spotify also weighs projected users. That is **unofficial and unverified** ([forum](https://community.spotify.com/forum/en/discussion/6995067/extension-request-not-letting-me)), and the official page says "Maintaining".

## 2. What a new app can do today (Development Mode)

| Rule | Value | Source |
|---|---|---|
| Users | **Up to 5** authenticated users, each added to an allowlist by Spotify email. Anyone not on the list gets **403** errors | [quota-modes](https://developer.spotify.com/documentation/web-api/concepts/quota-modes) |
| Owner | The owner needs **Premium**, and the app stops working if Premium lapses | [Feb 2026 migration guide](https://developer.spotify.com/documentation/web-api/tutorials/february-2026-migration-guide) |
| Apps per developer | 25, raised from 1 in July 2026. **The quota is shared across a developer account**, so extra apps don't add capacity | [July 2026 blog](https://developer.spotify.com/blog/2026-07-23-web-api-quota-updates), [July 2026 changelog](https://developer.spotify.com/documentation/web-api/references/changes/july-2026) |
| Quota | Dev Mode has a quota that is separate from rate limits. When you exceed it you get `429` with `"reason": "QUOTA_EXCEEDED"`. Spotify doesn't publish the numbers ("subject to change") | [quota-modes](https://developer.spotify.com/documentation/web-api/concepts/quota-modes) |
| Rate limit | A rolling 30-second window, lower in Dev Mode. **No published numbers** | [rate-limits](https://developer.spotify.com/documentation/web-api/concepts/rate-limits) |
| Intended use | "learning, experimentation, and personal projects for non-commercial use by individual developers … should not be relied on as a foundation for building or scaling a business" | [Feb 2026 blog](https://developer.spotify.com/blog/2026-02-06-update-on-developer-access-and-platform-security) |

Several more user caps on top of the 5-user limit would not help. The quota is per developer account, the policy requires "a separate Security Code for each SDA" and forbids "more than one Security Code for each SDA" ([policy VII.2](https://developer.spotify.com/policy)), and splitting one app across client IDs is **not a path**.

### Endpoints that still work in Dev Mode
These are taken from the "Endpoints still available" list in the [Feb 2026 changelog](https://developer.spotify.com/documentation/web-api/references/changes/february-2026).

- **Top artists and tracks:** `GET /me/top/{artists|tracks}`, with short (~4 weeks), medium (~6 months) and long (~1 year) ranges, up to 50 items ([ref](https://developer.spotify.com/documentation/web-api/reference/get-users-top-artists-and-tracks)).
- **Recently played:** `GET /me/player/recently-played`, up to 50 items ([ref](https://developer.spotify.com/documentation/web-api/reference/get-recently-played)).
- **Library:**
  - `GET /me/playlists`, `/me/tracks`, `/me/albums` and `/me/following` (followed artists).
  - `GET /playlists/{id}/items` works **only for playlists the user owns or collaborates on**.
- **Profile:** `GET /me`.
- **Metadata:** single-item gets only (`/artists/{id}`, `/tracks/{id}` and so on), and `/search` with a **maximum limit of 10**.
- **Player:** player and queue endpoints, which are Premium-oriented.

### Removed or restricted for new and Dev Mode apps
- **Since Nov 27, 2024:**
  - Related Artists, **Recommendations** and **Audio Features**.
  - Audio Analysis, Featured Playlists and Category Playlists.
  - 30-second previews in multi-get responses.
  - Algorithmic and editorial playlists.
  - Source: [blog](https://developer.spotify.com/blog/2024-11-27-changes-to-the-web-api). The reference pages now mark Recommendations and Audio Features "Deprecated" ([recs](https://developer.spotify.com/documentation/web-api/reference/get-recommendations), [audio features](https://developer.spotify.com/documentation/web-api/reference/get-several-audio-features)).
- **Since the February 2026 Dev Mode changes:**
  - **Batch gets:** `/tracks` and `/artists` can no longer fetch several IDs at once.
  - **Discovery:** artist top tracks, new releases and browse categories are gone.
  - **Other users:** other users' profiles and playlists (`/users/{id}`, `/users/{id}/playlists`) are gone.
  - **Removed fields:**
    - `popularity` on tracks, artists and albums, and artist `followers`.
    - On the user object: `email`, `country`, `product` and `followers`.
  - Source: [migration guide](https://developer.spotify.com/documentation/web-api/tutorials/february-2026-migration-guide).
  - **Status of these changes:** the blog's Mar 9 update **postponed the endpoint changes for apps that already existed**. They still apply to apps created after Feb 11, 2026, which includes any app Fuse makes now ([blog update](https://developer.spotify.com/blog/2026-02-06-update-on-developer-access-and-platform-security)).
- **Extended Quota apps** are "not affected by any of the changes" in the 2026 guide ([migration guide](https://developer.spotify.com/documentation/web-api/tutorials/february-2026-migration-guide)). Unverified: whether a **newly approved** extended app gets back the endpoints removed in 2024. That blog says only that apps with *existing* extended access are unaffected.

**What this means for Fuse:** top artists, top tracks, recently played and the user's own playlists give you a decent taste signal. You lose audio features, recommendations and popularity, so vibe and energy scores have to come from another source or from your own logic. Artist `genres` isn't on the removed-fields list, but I **didn't verify** that it is still populated.

## 3. Ways to get more access before 250k MAU

| Path | Status | Fuse eligible? |
|---|---|---|
| Extended Quota partner form | This is the only path to more than 5 users. You need an organization, a launched service, 250k MAU, key markets and commercial viability ([quota-modes](https://developer.spotify.com/documentation/web-api/concepts/quota-modes)) | **No, not yet** |
| Startup or early-access program | **Not found** on developer.spotify.com | n/a |
| Scope extension request | Exists only for apps that already have extended quota: "If you need to request additional scopes, you can notify Spotify by submitting a scope extension request" ([policy VII.3](https://developer.spotify.com/policy)) | No |
| "Open Access" scopes (`user-soa-*`, `soa-*`) | These appear in the [scopes list](https://developer.spotify.com/documentation/web-api/concepts/scopes) for partner account linking. It looks like a separate partner program for publishers and entitlements. **I didn't verify it and it doesn't seem relevant** | Unlikely |
| Spotify for Artists or Ads APIs | Not relevant, because they don't expose listener taste data. **Not researched in depth** | No |
| Multiple apps, sharing accounts, scraping, librespot or web-player internals, fake MAU | Breaks the policy and Developer Terms (for example, "using any robot, spider … to retrieve … Spotify Content", [terms IV](https://developer.spotify.com/terms)). **Not a path** | — |

The [community forum](https://community.spotify.com/forum/en/discussion/6920661/updating-the-criteria-for-web-api-extended-access) is the only official feedback channel Spotify names. Posting there won't get you access.

## 4. Data by scope, and the terms that matter

| Scope | Data | Fuse use |
|---|---|---|
| `user-top-read` | Top artists and tracks | Core taste signal |
| `user-read-recently-played` | The last 50 plays | Current mood |
| `playlist-read-private` / `-collaborative` | The user's own playlists | Optional |
| `user-follow-read` | Followed artists | Optional |
| `user-library-read` | Saved tracks and albums | Optional |
| `user-read-private` / `user-read-email` | Profile. In Dev Mode, `email` and `country` are removed | Skip. Use Firebase Auth instead |

Source for all scopes: [scopes](https://developer.spotify.com/documentation/web-api/concepts/scopes). Request only what you need: "Only request and process the data that is needed" ([policy I.1](https://developer.spotify.com/policy)).

The Developer Terms are [Version 10, effective May 15 2025](https://developer.spotify.com/terms), and the Developer Policy is [effective May 15 2025](https://developer.spotify.com/policy). These are the clauses that affect Fuse:

- ⚠️ **No user profiling or analytics.** "Do not analyze the Spotify Content or the Spotify Service for any purpose, including … creating new or derived listenership metrics … user metrics, or building profiles of users" ([policy III.13](https://developer.spotify.com/policy)). Turning a group's top artists into a "group taste profile" could fall under this. **This is the biggest terms risk, and it needs counsel or Spotify's view.**
- ⚠️ **No "commercial product offers".** "Do not generate news media, or commercial product offers" ([policy III.12](https://developer.spotify.com/policy)). Unverified how this applies to recommending venues. If venues ever pay for placement, the risk goes up.
- ⚠️ **No AI or ML.** You can't use Spotify Content "to train a machine learning or AI model or otherwise ingest Spotify Content into a machine learning or AI model" ([terms IV](https://developer.spotify.com/terms), [policy III.14](https://developer.spotify.com/policy)). **Don't pass Spotify data to an LLM to rank plans.**
- **Storage:**
  - Don't store, aggregate or compile Spotify Content "other than as strictly necessary to operate your SDA … Do not store Spotify Content indefinitely". Keep data fresh and delete older data.
  - Local caching is limited to temporary caching of metadata and cover art.
  - Source: [terms IV](https://developer.spotify.com/terms).
- **Personal data:**
  - Process it "for as long as is necessary".
  - Use beyond "displaying it back to the user" needs explicit consent for categories that weren't selected at connection.
  - Never sell it.
  - Source: [terms V](https://developer.spotify.com/terms).
- **Disconnect:** you need an easy, visible way to unlink, and you must **delete** that user's Spotify data when they unlink ([terms V](https://developer.spotify.com/terms)).
- **Required legal docs:** a privacy policy shown before signup, and an end-user agreement with Spotify-specific clauses, including that Spotify is a third-party beneficiary ([terms V](https://developer.spotify.com/terms)).
- **No ad tech:** you can't transfer Spotify data (including aggregate or derivative data) to any ad network or data broker, "even if a user consents" ([terms IV](https://developer.spotify.com/terms)).
- **Attribution:**
  - Any Spotify content you display must be attributed with Spotify marks.
  - Metadata and cover art must link back to Spotify.
  - The app name can't start with "Spot".
  - Source: [policy II.4, VI](https://developer.spotify.com/policy).
- **Other rules:**
  - The app can't be targeted at children ([policy III.8](https://developer.spotify.com/policy)).
  - Spotify can revoke unused permissions after 90 days ([terms](https://developer.spotify.com/terms)).
- **Commercial use:** this is OK for a **Non-Streaming SDA**, which is what Fuse would be. Ads, sponsorships and paid access are allowed ([policy IV.3](https://developer.spotify.com/policy)).

**Possible blocker for group plan picking:** III.13 (profiling) and III.12 (commercial offers). Showing "you both love Artist X" back to the users who linked is the safer pattern. Deriving scores that feed venue ranking is the riskier one. **This is my reading, not legal advice.**

## 5. Expo / React Native setup (high level)

- **Flow:** Spotify requires **Authorization Code with PKCE** for mobile and SPA clients. Implicit grant is deprecated, and HTTP redirects are dropped except for loopback `127.0.0.1` ([Feb 2025 blog](https://developer.spotify.com/blog/2025-02-12-increasing-the-security-requirements-for-integrating-with-spotify), [PKCE tutorial](https://developer.spotify.com/documentation/web-api/tutorials/code-pkce-flow)).
- **Redirect URI:**
  - It must match exactly, use HTTPS, and `localhost` isn't allowed ([redirect URIs](https://developer.spotify.com/documentation/web-api/concepts/redirect_uri)).
  - Custom schemes still work, but Spotify recommends Universal Links and App Links ([Feb 2025 blog](https://developer.spotify.com/blog/2025-02-12-increasing-the-security-requirements-for-integrating-with-spotify)).
  - For the web build, register `https://fuse-poc.web.app/...`.
- **Library:** `expo-auth-session`, which is official Expo with `usePKCE` on by default ([docs](https://docs.expo.dev/versions/latest/sdk/auth-session/)). Store any client-side values in `expo-secure-store` ([docs](https://docs.expo.dev/versions/latest/sdk/securestore/)).
- **Token exchange:** the PKCE token call needs only `client_id`, `code`, `redirect_uri` and `code_verifier`, with no secret ([PKCE tutorial](https://developer.spotify.com/documentation/web-api/tutorials/code-pkce-flow)).
  - **Suggested approach (my design choice, not a Spotify requirement):** send the code and verifier to a Firebase Function. The Function exchanges them, stores the **refresh token server-side** (encrypted), and makes all Spotify calls.
  - The refresh response "might not include a new refresh token". When it doesn't, keep the old one.
- **Client ID:** it's a "Security Code" that must be kept confidential ([terms VI](https://developer.spotify.com/terms)). Put it in env vars or Secret Manager, not committed literals.
- **Data model:** store minimal derived data (top artist IDs and names, refreshed periodically) with a TTL. Delete it on unlink. Handle `403` (not allowlisted) and `429` (check the `reason` field).
- **Unverified:** no official Spotify React Native SDK for Web API auth was checked. Community wrappers exist but aren't evaluated here.

## 6. Legitimate alternatives

| Option | Access requirement | Taste data | Commercial use | Fit |
|---|---|---|---|---|
| **First-party onboarding** (pick genres, artists, vibes) | None | Whatever you ask | Yours | **Best for launch** |
| **Apple Music API / MusicKit** | A media identifier and private key from Apple's Certificates, Identifiers & Profiles. That likely means the $99/yr Developer Program, which Fuse needs for iOS anyway ([MusicKit](https://developer.apple.com/musickit/), [programs](https://developer.apple.com/programs/)). **No MAU gate found** (unverified that none exists) | Recently played, **Apple Music Replay** (top songs, albums, artists), personalized recommendations ([MusicKit](https://developer.apple.com/musickit/)) | Apple Music API terms **not reviewed**. Unverified | Good optional link. Works on iOS, Android and web per Apple |
| **Last.fm API** | A free API key. Public user stats need only a username, with no OAuth (`user.getTopArtists`, periods from 7 days to overall) ([ref](https://www.last.fm/api/show/user.getTopArtists)) | Top artists and tracks, scrobbles | **Non-commercial only**, unless you get a commercial agreement via partners@last.fm ([API ToS 3.1–3.2](https://www.last.fm/api/tos)) | Possible, but you need the commercial deal first. Only covers users who scrobble |
| **ListenBrainz** | Open API. User stats by username ([stats API](https://listenbrainz.readthedocs.io/en/latest/users/api/statistics.html)) | Top artists, releases and recordings | Pre-revenue startups are asked to sign up for a **commercial support tier** ([MetaBrainz](https://metabrainz.org/supporters/account-type)) | Niche user base |
| **YouTube Music** | **No official YouTube Music API found in this pass** (unverified). Unofficial `ytmusicapi`-style libraries are not ToS-safe | — | — | Skip |
| Pasting a Spotify profile or playlist link | Dev Mode removed other users' profiles and playlists, and playlist items return only for the user's own playlists ([migration guide](https://developer.spotify.com/documentation/web-api/tutorials/february-2026-migration-guide)) | Very little | — | **Not useful** |

## 7. Architecture for Fuse now

**Go / no-go:**
- **No-go** on public Spotify linking for launch.
- **Go** on a 5-user internal prototype.
- **Go** on a first-party taste step.

**Build now:**
1. **Provider-agnostic `TasteProfile` table in SQL Connect:**
   - Columns: `source` (manual / spotify / apple), `artists[]`, `genres[]`, `updated_at` and `expires_at`.
   - The rest of the app reads only this table.
2. **Manual taste picker** in onboarding, which is the launch default.
3. **Spotify prototype** (Dev Mode, owned by the company's Premium account, allowlist of ≤5 team members):
   - Expo AuthSession with PKCE, then the Function does the token exchange.
   - Request only the scopes `user-top-read` and `user-read-recently-played`.
   - Unlink deletes the data.
4. Optional **Apple Music linking** through the same table.
5. **Legal prep:**
   - A privacy policy and EULA with Spotify's required clauses.
   - A lawyer's read on policy III.12 and III.13.
   - No Spotify data goes to any AI model.

**Wait on:**
- The Extended Quota application, until Fuse is launched, incorporated and approaching 250k MAU.
- Any feature that needs Recommendations or Audio Features, which are unavailable to new apps.

**Risks:**
- Extended Quota denial, or a wait of up to 6 weeks or longer.
- The Dev Mode 5-user cap and an unpublished quota.
- Spotify can further restrict Dev Mode at any time. The Feb 2026 blog says it will "continue to evolve this model".
- Enforcement for profiling or commercial-offer violations can include revoked keys and forced data deletion ([terms](https://developer.spotify.com/terms)).

## Unverified / still open
- Whether using Dev Mode to prototype a company product is OK, given that Spotify says Dev Mode is for "non-commercial use by individual developers". That points to low risk for internal tests and high risk for public users.
- Actual Dev Mode quota and rate-limit numbers. They're unpublished.
- Whether projected MAU counts toward 250k. Only a forum moderator has said so.
- Whether a newly granted Extended Quota app regains the endpoints removed in 2024.
- Whether artist `genres` is still populated for new Dev Mode apps.
- How III.12 and III.13 apply to group taste matching for venues. This needs a legal read.
- Apple Music API terms on storage and commercial use, and whether any usage gate exists.
- What Spotify Open Access scopes are and who they're for.
- Whether there is an official YouTube Music API.
- No Spotify changelog after July 2026 was found. The August to November 2026 changelog URLs returned 404 on 2026-11-12.

---

## Claude Code prompt (read-only planning)

```text
You're in C:\Users\User\fuse (Windows/PowerShell, Expo/React Native + Firebase: Auth, SQL Connect on Postgres, Functions, Hosting, Secret Manager). Read research/spotify-api-2026-11-12.md in C:\Users\User\fuse\fuse-research first if present.

PLANNING ONLY. Do not modify any files, install packages, or change package.json, app.json/app.config, Firebase config, SQL Connect schema, or Functions. Ask me before any change and before any git commit. Never write secrets or client IDs as literal values in files: they live only in Windows user env vars (e.g. [Environment]::SetEnvironmentVariable("SPOTIFY_CLIENT_ID","...","User")) or Firebase Secret Manager, referenced as ${VAR}.

Context: Spotify Web API new apps are in Development Mode: max 5 allowlisted users, owner needs Premium, Recommendations/Audio Features/popularity unavailable, batch gets removed, search limit 10. Extended quota needs a registered org + launched service + 250k MAU. Spotify policy bans building user profiles/derived user metrics (III.13), commercial product offers (III.12), and feeding Spotify data into any AI/ML model; data must be deleted on unlink.

Do this:
1. Inspect the app: auth setup, navigation/onboarding, deep-link scheme, Expo SDK version, whether we use a dev build, existing Functions, and SQL Connect schema.
2. Draft a plan (in chat, no edits) for:
   a. A provider-agnostic TasteProfile model (source: manual|spotify|apple, artists, genres, updated_at, expires_at) and how the group-plan flow would read it.
   b. A manual taste-picker onboarding step (the launch default).
   c. A Spotify Dev Mode prototype: expo-auth-session with PKCE, redirect URI choices for iOS/Android/web (https://fuse-poc.web.app), code+verifier sent to a Firebase Function that exchanges tokens, stores the refresh token encrypted server-side, requests only user-top-read and user-read-recently-played, handles 403 (not allowlisted) and 429 (reason QUOTA_EXCEEDED), and deletes all Spotify data on unlink.
   d. Confirm no Spotify data would reach any AI/LLM code path; flag any place it might.
3. List the exact packages, config, and env vars that would be needed, and the files you'd touch.
4. Stop and wait for my yes before implementing anything.
```
