# Fuse B2B: MCP servers + GitHub repos (2026-11-13)

This report covers concrete MCP servers and repos for building Fuse's ad, analytics and media-management platform for local businesses.
**Stack:** Expo/RN · Firebase (Auth, SQL Connect / Cloud SQL Postgres, Functions, Hosting) · Windows + Claude Code.

**How each fact was checked (2026-11-13):**
- **Stars, license and last push** come from the GitHub API today.
- **"Release"** is GitHub's latest *release*. "none" means the repo publishes no GitHub releases.
- **Hosted MCP endpoints** were probed today with an MCP `initialize` call. A 401 is an OAuth or key challenge, which means the server is live.
- **Dead or archived** entries were dropped. The old repos `PostHog/mcp` and `tinybirdco/mcp-tinybird` are **archived**, so use the hosted servers instead.
- **License caution:** AGPL means that if you modify the code and serve it to users, you must publish your changes. GPL means copyleft if you distribute the code.

**Covered earlier (business-tools report or previous scans):**
- The Stripe, PostHog, Cloudinary, Mux, HubSpot, Attio, Ayrshare and Firebase MCPs, plus react-admin, Refine, Metabase, Cube and Tinybird, are repeated here only with their repo and install line, tagged *(earlier)*.
- RevenueCat AI Toolkit, OneSignal MCP, Linear MCP, Resend MCP, GitHub MCP, Apify, Scrapling, Parallel MCP, Mapbox and the Google Maps MCPs are already covered and skipped.

---

## Top 10 (ranked for Fuse)

| # | Item | Type | Why it's top | Install |
|---|---|---|---|---|
| 1 | **Stripe plugin + MCP** *(earlier)*: https://github.com/stripe/ai · `https://mcp.stripe.com` | Official. MIT. 2,059★. Pushed 2026-11-13. No GitHub releases | Ad budgets, billing and refunds from Claude | `claude plugin install stripe@claude-plugins-official` (OAuth) |
| 2 | **Firebase plugin/MCP** *(earlier)*: https://github.com/firebase/firebase-tools | Official. MIT. 4,489★. v15.36.0 (2026-11-11) | Auth custom claims (orgs/roles), SQL Connect, Functions logs, Storage | `claude plugin marketplace add firebase/firebase-tools` → `claude plugin install firebase@firebase` |
| 3 | **MCP Toolbox for Databases**: https://github.com/googleapis/mcp-toolbox | Official Google. Apache-2.0. 16,857★. v1.17.0 (2026-11-13) | Lets Claude run SQL on **Cloud SQL Postgres** (ad_events, rollups) and BigQuery | `npx -y @toolbox-sdk/server --prebuilt=postgres --stdio` (connection env vars per the prebuilt docs) |
| 4 | **PostHog plugin + MCP** *(earlier)*: https://github.com/PostHog/ai-plugin · `https://mcp.posthog.com/mcp` | Official. No license file detected. 100★. Pushed 2026-11-13 | Product analytics, flags and errors from Claude | `claude plugin marketplace add PostHog/ai-plugin` → `claude plugin install posthog@posthog` |
| 5 | **Meta Ads MCP (official, hosted)**: `https://mcp.facebook.com/ads` · docs https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-mcp-server/ads-mcp-server-overview | Official Meta, hosted (no repo). Open beta from Apr 29, 2026; own-app access from Jul 16, 2026 | Run Fuse's own growth ads now. Later, a "boost this plan card to Instagram/Facebook" feature for businesses. | `claude mcp add --transport http meta-ads https://mcp.facebook.com/ads` → `/mcp` |
| 6 | **Cloudinary MCP servers** *(earlier)*: https://github.com/cloudinary/mcp-servers | Official. MIT. 10★. Pushed 2026-11-04 | Media library per business, transforms, moderation/analysis, AI image generation | `/plugin install cloudinary@claude-plugins-official` (OAuth) |
| 7 | **Postiz**: https://github.com/gitroomhq/postiz-app | Community/OSS. **AGPL-3.0**. 41,524★. v2.25.3 (2026-10-24) | Self-hosted social scheduler. Use it as a reference or a sidecar for "post your promo everywhere" | `git clone https://github.com/gitroomhq/postiz-app` |
| 8 | **react-admin** *(earlier)*: https://github.com/marmelab/react-admin | OSS. MIT. 26,992★. v5.15.7 (2026-11-13) | Internal admin: approve businesses, moderate creatives, refunds | `npm create react-admin@latest fuse-admin` |
| 9 | **Google Ads MCP (official)**: https://github.com/googleads/google-ads-mcp | Official Google. Apache-2.0. 1,170★. v0.0.6 (2026-11-02) | Fuse's own Google/YouTube acquisition ads. Changes go through propose-then-confirm with a checksum, and a read-only mode is available | `pipx run --spec git+https://github.com/googleads/google-ads-mcp.git google-ads-mcp` (Application Default Credentials) |
| 10 | **marketingskills**: https://github.com/coreyhaines31/marketingskills | Community. MIT. 60,119★. v2.14.12 (2026-11-11) | Claude skills for Joe: CRO, ad copy, analytics, launch, pricing | `claude plugin marketplace add coreyhaines31/marketingskills` → `claude plugin install marketing-skills@marketingskills` |

---

## 1. Ads

| Item | Official? | License · ★ · release/push | What it does for Fuse | Install |
|---|---|---|---|---|
| Meta Ads MCP `https://mcp.facebook.com/ads` (live, 401) | Official, hosted | n/a | Ad reporting and campaign management for accounts you authorize. To manage *other businesses'* ad accounts from your own app, read Meta's docs first: app-access requirements were not verified this run | `claude mcp add --transport http meta-ads https://mcp.facebook.com/ads` |
| googleads/google-ads-mcp | Official | Apache-2.0 · 1,170★ · v0.0.6 2026-11-02 | Google Ads reporting plus guarded writes | See Top 10 #9. Set `labels: false` and `assets: false` in `tools_config.yaml` for read-only |
| pipeboard-co/meta-ads-mcp | Community | NOASSERTION (custom) · 1,367★ · 1.0.128 2026-09-23 | Older community Meta Ads MCP. Prefer Meta's official server | Per repo README |
| amekala/ads-mcp | Community | MIT · 107★ · pushed 2026-11-03 | One MCP for Google, Meta, **LinkedIn and TikTok** Ads. I found no official LinkedIn or TikTok Ads MCP repos | Per README |
| markifact/markifact-mcp | Community | MIT · 60★ · pushed 2026-11-10 | Google, Meta, GA4, TikTok and LinkedIn Ads | Per README |
| revive-adserver/revive-adserver | OSS | **GPL-2.0** · 1,520★ · no GH releases, pushed 2026-11-11 | Classic PHP banner/zone ad server. **Reference only.** It's the wrong shape for sponsored plan cards in a swipe deck | `git clone https://github.com/revive-adserver/revive-adserver` |
| kijepark/adserver-tutorial | Community | MIT · 114★ · pushed 2024-03-20 (stale) | Node.js ad-server tutorial covering selection, tracking and pacing ideas. Read it, don't depend on it | `git clone` |
| ⚠️ irinabuht12-oss/google-ads-meta-ads-mcp | Community | MIT · 7,551★ · pushed 2026-10-27 | **Not recommended.** It has a very high star count for an unknown author, and its sibling skills repo has no license. I couldn't verify where it came from | — |

**Fuse takeaway:** there's no good open-source sponsored-listing engine for this use case. Keep the in-house ranker (see the earlier report). The ad MCPs above are for **Fuse's own growth ads** and for a later "boost to Meta/Google" upsell.

## 2. Analytics

| Item | Official? | License · ★ · release | What it does for Fuse | Install |
|---|---|---|---|---|
| googleapis/mcp-toolbox | Official | Apache-2.0 · 16,857★ · v1.17.0 2026-11-13 | SQL on Postgres, Cloud SQL, BigQuery and Firestore from Claude. Use it to inspect `ad_events` and build rollups | Top 10 #3 |
| PostHog MCP `https://mcp.posthog.com/mcp` (401) *(earlier)* | Official | — | Product analytics | `claude mcp add --transport http posthog https://mcp.posthog.com/mcp` |
| Amplitude MCP `https://mcp.amplitude.com/mcp` (401) + https://github.com/amplitude/mcp-marketplace | Official | MIT · 47★ · pushed 2026-11-13 | Alternative to PostHog | `claude mcp add --transport http amplitude https://mcp.amplitude.com/mcp` or `claude plugin install amplitude` |
| Mixpanel MCP `https://mcp.mixpanel.com/mcp` (401) + https://github.com/mixpanel/ai-plugins | Official | Apache-2.0 · 18★ · pushed 2026-11-06 | Alternative to PostHog. Service-account header option `Bearer Basic ${MIXPANEL_SA_TOKEN}` | `claude plugin marketplace add mixpanel/ai-plugins` → `claude plugin install mixpanel` |
| googleanalytics/google-analytics-mcp | Official | Apache-2.0 · 3,776★ · 1.1.0 2026-11-06 | GA4 reports (Firebase Analytics data) | `claude mcp add analytics-mcp -- pipx run analytics-mcp` (needs ADC with the analytics.readonly scope) |
| umami-software/umami | OSS | **MIT** · 40,119★ · v3.5.0 2026-11-12 | Self-host web analytics. Could power a "your website traffic" add-on for businesses | `git clone https://github.com/umami-software/umami` |
| Openpanel-dev/openpanel | OSS | AGPL-3.0 · 7,296★ · no GH releases, pushed 2026-11-09 | Open-source Mixpanel-style product analytics plus web analytics | `git clone` |
| plausible/analytics | OSS | AGPL-3.0 · 29,840★ · v3.2.1 2026-05-15 | Privacy web analytics | `git clone` |
| matomo-org/matomo | OSS | GPL-3.0 · 22,028★ · pushed 2026-11-13 | Full GA alternative (PHP) | `git clone` |
| PostHog/posthog | OSS | NOASSERTION (mixed MIT/EE) · 40,773★ | Self-host option | — |
| cube-js/cube *(earlier)* | OSS core | NOASSERTION · 21,224★ · v1.8.23 2026-11-13 | Semantic layer and metrics API | `npx cubejs-cli create` (per Cube docs) |
| metabase/metabase *(earlier)* | OSS | NOASSERTION (AGPL core + EE) · 50,313★ · v0.65.4 2026-11-13 | Internal BI on Postgres | Docker per docs |
| **Charts** recharts/recharts | OSS | MIT · 27,710★ · pushed 2026-11-13 | Advertiser dashboard charts on Expo **web** | `npm i recharts` |
| **Charts** FormidableLabs/victory-native-xl | OSS | GitHub API detects no license (check the repo) · 1,236★ · victory-native@43.0.1 2026-10-20 | Native RN charts (Skia), for the in-app dashboard | `npm i victory-native` (+ Skia, Reanimated per docs) |
| **Charts** apache/echarts | OSS | Apache-2.0 · 67,716★ · pushed 2026-10-13 | Heavy-duty web charts | `npm i echarts` |
| tremorlabs/tremor | OSS | Apache-2.0 · 3,700★ · last push 2025-10-10 (quiet) | Copy-paste dashboard components (web) | — |

## 3. Media management + social

| Item | Official? | License · ★ · release | What it does for Fuse | Install |
|---|---|---|---|---|
| cloudinary/mcp-servers *(earlier)* | Official | MIT · 10★ · pushed 2026-11-04 | Asset management, environment config, metadata, analysis, image generation | Top 10 #6 |
| **ImageKit MCP** (hosted): DAM `https://imagekit.io/mcp/dam` (401), Admin `https://imagekit.io/mcp/admin`, DevTools `https://devtools-mcp.imagekit.io/mcp` (200, no auth) · https://github.com/imagekit-developer/skills | Official | skills: MIT · 2★ | Cheaper alternative to Cloudinary, with a DAM | `/plugin marketplace add imagekit-developer/skills` → `/plugin install imagekit-plugin@imagekit-tools` |
| Mux MCP `https://mcp.mux.com` (401) *(earlier)* | Official, hosted | — | Video plus Robots AI | `claude mcp add --transport http mux https://mcp.mux.com` |
| Ayrshare plugin *(earlier)*: https://github.com/ayrshare/ayrshare-social-media-api-claude-plugin · `https://api.ayrshare.com/mcp` | Official | No license file · 21★ · pushed 2026-11-10 | Paid social posting API with an MCP | `claude plugin marketplace add ayrshare/ayrshare-social-media-api-claude-plugin` → `claude plugin install ayrshare@ayrshare` |
| gitroomhq/postiz-app | OSS | **AGPL-3.0** · 41,524★ · v2.25.3 2026-10-24 | Self-host social scheduler. Free alternative to Ayrshare *for internal or Joe use*. Embedding it in Fuse triggers AGPL | `git clone` / Docker |
| inovector/mixpost | OSS | MIT · 3,930★ · 2.6.0 2026-03-16 | Laravel social scheduler. MIT is friendlier to embed | `composer` per README |
| transloadit/uppy | OSS | MIT · 31,068★ · uppy@6.0.7 2026-11-05 | Upload UI (resumable, webcam, dashboard) for the business media library on **web** | `npm i uppy` |
| directus/directus | OSS | NOASSERTION (BSL-style) · 38,711★ · v12.6.1 2026-11-04 | Headless CMS/DAM over **your Postgres**, with roles, files and an API | `npx directus init` |
| payloadcms/payload | OSS | MIT · 45,804★ · v3.93.0 2026-11-04 | Next.js CMS with uploads, access control, multi-tenant plugin | `npx create-payload-app` |
| immich-app/immich | OSS | AGPL-3.0 · 119,091★ · pushed 2026-11-13 | Photo/video management. **Reference UX only.** It's a personal-photo app, not a B2B DAM | — |

## 4. B2B SaaS plumbing

| Item | Official? | License · ★ · release | What it does for Fuse | Install |
|---|---|---|---|---|
| stripe/ai *(earlier)* | Official | MIT · 2,059★ | Stripe plugin, skills, agent toolkit | Top 10 #1; skills via `npx skills add https://docs.stripe.com` |
| getlago/lago | OSS | **AGPL-3.0** · 10,830★ · v1.57.0 2026-11-05 | Open-source usage metering and billing (bill per impression/plan) | `git clone` / Docker |
| openmeterio/openmeter | OSS | Apache-2.0 · 2,444★ · v1.0.0-beta.236 2026-10-09 | Usage metering. Pairs with Stripe | `git clone` |
| marmelab/react-admin *(earlier)* | OSS | MIT · 26,992★ · v5.15.7 | Admin panel | Top 10 #8 |
| refinedev/refine *(earlier)* | OSS | MIT · 36,035★ · @refinedev/core@5.0.12 2026-04-02 | Admin/B2B framework | `npm create refine-app@latest` |
| boxyhq/saas-starter-kit | OSS | Apache-2.0 · 4,977★ · v1.6.0 2024-12-24 (older) | Reference for **teams/orgs, roles, invites, audit log, webhooks** | `git clone` (read the org/team model) |
| nextjs/saas-starter | OSS | MIT · 16,251★ · pushed 2025-12-11 | Postgres + Stripe + teams reference | `git clone` |
| ixartz/SaaS-Boilerplate | OSS | MIT · 7,489★ · pushed 2026-11-01 | Multi-tenant org + roles reference (Next.js) | `git clone` |
| twentyhq/twenty | OSS | NOASSERTION (AGPL core) · 60,882★ · twenty/v2.58.0 2026-11-13 | Self-host CRM, an alternative to HubSpot/Attio | Docker |
| HubSpot MCP `https://mcp.hubspot.com` (401) + https://github.com/HubSpot/hubspot-mcp-plugins *(earlier)* | Official | Apache-2.0 · 5★ · pushed 2026-10-12 | Joe's CRM, social and campaigns | `/install-plugin https://github.com/HubSpot/hubspot-mcp-plugins` |
| Attio MCP `https://mcp.attio.com/mcp` (401) *(earlier)* | Official, hosted | — | CRM. OAuth, on every plan | `claude mcp add --transport http attio https://mcp.attio.com/mcp` |
| casdoor/casdoor | OSS | Apache-2.0 · 14,843★ · pushed 2026-11-11 | IAM/SSO with orgs, as an alternative to Identity Platform. Only needed for enterprise SSO | — |

## 5. Claude Code plugins / skills (marketing, ads, analytics)

| Item | License · ★ · release | Use | Install |
|---|---|---|---|
| coreyhaines31/marketingskills | MIT · 60,119★ · v2.14.12 2026-11-11 | CRO, copy, SEO, analytics, launch, pricing | Top 10 #10 |
| anthropics/claude-plugins-official | Apache-2.0 · 47,148★ · pushed 2026-11-13 | Official plugin directory (Stripe, Cloudinary and others) | `/plugin` browse |
| anthropics/skills | No license file at the repo root · 194,047★ · pushed 2026-11-06 | Official Agent Skills examples | `/plugin marketplace add anthropics/skills` |
| PostHog/ai-plugin *(earlier)* | 100★ | Analytics skills + MCP | Top 10 #4 |
| mixpanel/ai-plugins · amplitude/mcp-marketplace | Apache-2.0 / MIT | Vendor analytics skills | See §2 |
| alirezarezvani/claude-skills | MIT · 29,751★ · pushed 2026-11-12 | 400+ mixed skills (some marketing). Cherry-pick, don't bulk-install | `npx skills add alirezarezvani/claude-skills` |
| appeeky/aso-skills | MIT · 2,298★ · pushed 2026-11-13 | App Store Optimization for Fuse's consumer app | `npx skills add appeeky/aso-skills` |

Security: skills and MCPs run with your permissions. Read third-party skills before installing them. NVIDIA/SkillSpector (Apache-2.0, 25,834★) scans skills for malicious patterns.

---

## Copy-paste Claude Code prompt (run from `C:\Users\User\fuse`)

```
Set up MCP servers and plugins for Fuse's B2B ads/analytics/media work. Background: C:\Users\User\fuse\fuse-research\research\business-mcps-repos-2026-11-13.md (read it first).

RULES (non-negotiable):
- Ask me before changing any app code, package.json, Firebase config (firebase.json, .firebaserc, dataconnect/, rules), and before any git commit.
- Prefer OAuth for every server. Where a secret is unavoidable, it lives ONLY in a Windows user environment variable and is referenced in .mcp.json as ${VAR}. Never write a literal key, password, or credentials path into any file, command, or chat. For env vars give me the PowerShell line [Environment]::SetEnvironmentVariable("NAME","<paste>","User") with a placeholder only.
- Project scope (.mcp.json at repo root). Show me the full .mcp.json diff and wait for my OK before saving. Use test/sandbox accounts and our POC Firebase project only.
- Read-only by default: when a server supports read-only mode or approval rules, enable them.

INSTALL (one at a time, confirm each):
1. Stripe: claude plugin install stripe@claude-plugins-official  (OAuth via /mcp, sandbox only)
2. Firebase: claude plugin marketplace add firebase/firebase-tools ; claude plugin install firebase@firebase
3. PostHog: claude plugin marketplace add PostHog/ai-plugin ; claude plugin install posthog@posthog
4. Cloudinary: tell me to run /plugin install cloudinary@claude-plugins-official (OAuth)
5. MCP Toolbox for Postgres (Cloud SQL, POC instance, read-only DB user): add stdio server "toolbox-postgres" = npx -y @toolbox-sdk/server --prebuilt=postgres --stdio. Look up the exact env var names in https://mcp-toolbox.dev/documentation/configuration/prebuilt-configs/ and reference them as ${VAR} (password especially). Tell me if I need the Cloud SQL Auth Proxy running. If the npx stdio server fails on Windows, try wrapping with cmd /c.
6. Meta Ads (official, hosted): claude mcp add --transport http --scope project meta-ads https://mcp.facebook.com/ads ; then /mcp to OAuth. Use reporting/read tools only. Do not create or edit campaigns.
7. Marketing skills: claude plugin marketplace add coreyhaines31/marketingskills ; claude plugin install marketing-skills@marketingskills
OPTIONAL (ask first): Google Ads MCP (googleads/google-ads-mcp via pipx + Application Default Credentials, read-only tools_config), Mixpanel or Amplitude instead of PostHog, ImageKit instead of Cloudinary, HubSpot or Attio CRM.

VERIFY: run claude mcp list and /mcp, report each server's status, and run one harmless read per server (e.g. Stripe account info, Firebase list projects, PostHog list projects, Toolbox list_tables). Do not create, update, or delete anything. Then stop and summarize what's connected and what still needs my login.
```
