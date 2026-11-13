# Fuse B2B tools: ads, analytics, media (2026-11-13)

Fuse is moving from a consumer group-plan swiping app to ad, analytics and media-management software for local businesses. Businesses run promoted plan cards, see their results and manage their content.

**Stack:** Expo/RN · Firebase (Auth, SQL Connect on Cloud SQL Postgres, Functions, Hosting, Secret Manager) · POC/Test/Prod
**Method:** I fetched every price below from the vendor's own page on 2026-11-13, unless the row says otherwise. Anything marked "n/p" isn't published, so I didn't guess it.
**Already have, not re-covered:** RevenueCat AI Toolkit, OneSignal MCP, Linear MCP, Resend MCP, GitHub MCP, Apify, Scrapling, Parallel MCP, Mapbox, Google Maps MCPs.
- Resend still handles transactional email: receipts, campaign alerts and weekly reports.
- Linear still tracks the build.

---

## TL;DR: recommended starter stack (ranked)

| # | Layer | Pick | Why | Starting cost |
|---|---|---|---|---|
| 1 | Billing | **Stripe Billing + Checkout** (+ Radar) | Prepaid ad budgets, subscriptions, usage meters, customer portal. Official MCP. | 2.9% + 30¢ per card charge; Billing 0.7% of Billing volume |
| 2 | Ad engine | **Build in-house** (Postgres tables + Functions) | Fuse's ad unit is one card in a swipe deck, so an enterprise ad server is overkill at Huntington scale | $0 beyond Firebase |
| 3 | Product analytics | **PostHog** | Free 1M events/mo, flags, replay, error tracking, free MCP | Free tier |
| 4 | Business dashboards | **Own Postgres rollups → charts in app** first; **Tinybird** when event volume grows | Advertisers only see their own data, and both options keep that tenant-scoped | $0, then Tinybird Developer $49/mo |
| 5 | Media | **Cloudinary** (images + upload + moderation add-ons); **Mux** if video | Official MCPs and Claude plugin; transforms + CDN | Cloudinary Free 25 credits/mo; Mux pay-as-you-go |
| 6 | Moderation | **Google Cloud Vision SafeSearch** (same GCP project) | First 1,000/mo free, $1.50/1,000 after | ~Free at launch |
| 7 | Multi-tenant orgs | **Postgres `orgs`/`memberships` + Firebase custom claims** | No new vendor | $0 |
| 8 | CRM (Joe) | **HubSpot Free** or **Attio Free** | Both free for small teams, both have official MCP | $0 |
| 9 | Support | **Crisp Free** | 2 seats free; chat SDKs | $0 |
| 10 | Admin panel | **react-admin** (MIT) on Firebase Hosting | Internal approvals/moderation/refunds | $0 |
| Later | Social scheduling | **Ayrshare** (Launch) | Post to 14+ networks for each business. Has an MCP and a Claude Code plugin. | $299/mo (28-day trial) |
| Later | Ad server | **Kevel** / **Topsort** | Only once there's real auction volume or many placements | Sales-quoted |

## Build order (short)
1. **Org model:** `orgs`, `memberships(role)`, `places(place_id)` in SQL Connect, plus a custom claim `{orgId, role}` that a Function sets. Admin approval goes through react-admin.
2. **Business onboarding:** claim a Google Place, then verify it (place phone/website domain email, and Stripe Identity only for disputed claims). Stripe Checkout then takes the first prepaid budget.
3. **Campaigns + serving:** `campaigns`, `creatives`, `budgets`, and a Function `getSponsoredCards(userCtx)`. It filters by geo, time and category, ranks by bid × quality, and paces spend daily. Promoted cards go into the deck at fixed slots (for example 1 in 8) with a "Sponsored" label.
4. **Event pipeline:** write `impression`, `swipe_right`, `save`, `plan_created` and `redeem` events server-side through a Function into an `ad_events` table, deduped per user per creative per day. App Check is enforced. Product events go to PostHog.
5. **Advertiser dashboard:** a nightly or hourly rollup (`ad_daily_stats`) feeds a dashboard in the Expo web build: reach, right-swipes, saves, plans made, redemptions, cost per plan.
6. **Media library:** uploads go to Cloudinary through signed uploads, SafeSearch screens them on upload, and approved assets are tagged per `orgId`.
7. **Billing maturity:** add Stripe Meters (CPM/CPC or per-plan) and credit grants, or stay on prepaid budgets. Add Radar.
8. **Later:** Ayrshare cross-posting, Tinybird for real-time stats, and Kevel/Topsort if volume justifies it.

---

## 1. Ad serving / self-serve ad platform

| Tool | What | Fit for Fuse | Cost / free tier | Maturity | Setup notes |
|---|---|---|---|---|---|
| **In-house (recommended v1)** | Campaign tables + ranking/pacing Function | One placement type (sponsored plan card), one town. You keep full UX control and avoid vendor minimums. | Firebase only | You own it | Rank = bid × predicted engagement × freshness. Include a daily budget pacer, frequency cap per user, and a "Sponsored" label. |
| **Kevel** | API-first ad server: Decision, Management and Reporting APIs, sponsored listings, first/second-price auctions, autobid, catalog, "auction-as-a-service" (send eligible IDs, get winners) | Good upgrade path because it can auction only the cards *your* deck logic picks | n/p: Kevel doesn't publish prices ("SaaS Pricing", custom quote). A third-party comparison (Epom, Aug 2026) says pricing is based on modules + API request volume | Mature (iFood, retail media networks) | https://kevel.com/ad-units/sponsored-listings · https://dev.kevel.com/docs/retail-media · https://kevel.com/blog/auction-as-a-service |
| **Topsort** | Retail-media auctions API: listings, banners, video, sponsored brands, geo targeting, quality scores | Same "you choose candidates, they run the auction" model; geolocation trigger fits local | n/p (sales) | Mature, marketplace-focused | https://docs.topsort.com/en/ad-server/auctions/auctions-api · Don't cache auction results; run one auction per page view |
| **Stripe Billing** | Subscriptions, Meters (usage), credit grants, customer portal, Smart Retries | Prepaid ad budget = credit grant; spend = metered usage; or a simple monthly "Promoted" plan | 0.7% of Billing volume, pay-as-you-go. Meters include up to 100M events/mo. Portal custom domain $10/mo. Cards 2.9% + 30¢, ACH 0.8% | Mature | https://stripe.com/billing/pricing · Credit grants apply only to **metered** subscription items, not one-off invoices: https://docs.stripe.com/billing/subscriptions/usage-based/billing-credits |
| **Stripe Checkout / Payment Links** | Hosted pay page | Fastest v1: "Add $100 budget" → webhook → `budgets` row | Card fees only | Mature | Webhook into a Function; secret key in Secret Manager |
| **Stripe Connect** | Pay *out* to businesses | **Not needed for selling ads.** Only needed if Fuse later takes payments for businesses (bookings, deals) | "Stripe handles pricing" model: no platform fees. "You handle pricing" model: $2 per monthly active account + 0.25% + 25¢ per payout | Mature | https://stripe.com/connect/pricing |
| **Stripe Radar** | Card fraud | Stops stolen-card budget top-ups | Standard from $10/mo, or pay as you go. Plus from $14/mo, Pro from $20/mo (platform tiers cost more) | Mature | https://stripe.com/radar/pricing |
| **Firebase App Check** | Attests requests come from your real app | Main defense against fake impressions/clicks. Needed before you bill per event | No-cost (quotas vary by provider) | Mature | https://firebase.google.com/pricing · Enforce it on the event-logging Function |
| **AppsFlyer** | Mobile attribution + Protect360 fraud | Only if Fuse buys paid installs. Attribution *for advertisers* is first-party (swipe → plan → redeem) | Zero / Growth / Enterprise plans exist; the prices weren't in the fetched page (n/p) | Mature | https://www.appsflyer.com/pricing/ |

**Attribution for advertisers (first-party, recommended):** track impression → right-swipe → plan created → plan attended or redeemed. A per-campaign **redeem code/QR** closes the loop in-store, and that's the metric local businesses will pay for.

## 2. Analytics

### 2a. Product/event analytics (Fuse's own)
| Tool | Free tier | Paid / startup | MCP | Notes |
|---|---|---|---|---|
| **PostHog** (pick) | 1M events, 5K recordings, 1M flag requests, 100K exceptions, 1,500 survey responses per month; free plan 1 project, 1-yr retention | Pay-as-you-go above that, with billing limits per product | Free hosted MCP `https://mcp.posthog.com/mcp`; `npx @posthog/wizard mcp add` | https://posthog.com/pricing · https://posthog.com/docs/model-context-protocol · Use separate projects per env (paid gives 6 projects) |
| **Amplitude** | 2M events/mo, 10K replays | Plus "starts at $0"; startups (<$10M funding, <20 staff) get 1 free year of Growth | "AI Agents and MCP" on all plans | https://amplitude.com/pricing |
| **Mixpanel** | 1M events/mo, 10K replays | Growth: 1.5M events $140/mo (monthly). Startups (<5 yrs, ≤$8M) first year free | MCP on all plans | https://mixpanel.com/pricing/ |
| **Firebase Analytics (GA4) + BigQuery export** | Analytics no-cost; BigQuery sandbox on Spark | BigQuery pricing on Blaze | (none verified) | Export is a **daily** sync capped at **1M events/day** (non-360), so it's not suitable for live advertiser dashboards. https://firebase.google.com/docs/projects/bigquery-export |

### 2b. Customer-facing (advertiser) dashboards
| Tool | What | Cost | Fit |
|---|---|---|---|
| **Own rollups in Cloud SQL** (pick v1) | `ad_events` → hourly `ad_daily_stats` per org/campaign, served by SQL Connect queries guarded by `orgId` | $0 extra | Simplest, tenant-safe, and fine for one town |
| **Tinybird** | Managed ClickHouse; turns SQL into authenticated API endpoints; Events API ingest; remote MCP | Free: 0.25 vCPU, 1k requests/day, 10 GB. Developer $49/mo (0.5 vCPU, 25 GB); overage $0.0002/vCPU-sec | Best when events reach millions. Use JWT-scoped endpoints per org. https://www.tinybird.co/pricing |
| **Cube** | Semantic layer + APIs + embedding | Free (1,000 requests/day, MCP). Starter $40/dev/mo. **Embedded dashboards need Premium, $80/dev/mo** | Good if you want one metrics definition across app, AI and BI. https://cube.dev/pricing |
| **Metabase** | BI + embedding | OSS free (self-host, guest embeds carry a "Powered by Metabase" badge, MCP). Starter $100/mo. **Multi-tenant embedding (row-level security, white-label) is Pro, $575/mo** | Great *internal* BI on Postgres today; customer embedding is expensive. https://www.metabase.com/pricing/ |
| **Evidence** | BI-as-code | Team $2,500/mo; embedding is **Enterprise only** | **Skip.** https://evidence.dev/pricing |

**Warehouse:** keep Postgres (Cloud SQL) as the system of record. Add Tinybird for high-volume events, or BigQuery if you need GA4 joins. Don't add Snowflake-class tools yet.

## 3. Media management
| Tool | What | Cost / free | MCP / Claude | Notes |
|---|---|---|---|---|
| **Cloudinary** (pick) | Upload widget, image and video transforms, CDN, auto-tagging, add-ons (moderation, AI image generation), Assets DAM | Free: 25 credits/mo, 3 users (1 credit = 1,000 transforms, 1 GB storage or 1 GB image bandwidth). Plus $89/mo, 225 credits. Advanced $224/mo | Official remote MCPs: asset management, environment config, structured metadata, analysis (moderation/tagging), MediaFlows. Claude plugin: `/plugin install cloudinary@claude-plugins-official`. Skills: `npx skills add cloudinary-devs/skills` | https://cloudinary.com/pricing · https://cloudinary.com/documentation/cloudinary_llm_mcp · Use signed uploads from a Function; folder per `orgId` |
| **ImageKit** | Same category as Cloudinary, cheaper entry | Forever free: 20 GB bandwidth, 3 GB DAM. Lite $9/mo + pay-as-you-go. Pro $89/mo | n/a verified | Budget alternative. https://imagekit.io/plans/ |
| **Mux** | Video upload/encode/stream/player + Mux Data + "Robots" AI (summaries, moderation, chapters, social copy) | No platform fee. 1080p basic input is free; storage $0.003/min/mo; 100,000 delivery min/mo free then $0.001/min; $20/mo usage credit on pay-as-you-go; 100K free Robots units/mo; startups can apply for $500 | Official MCP `https://mcp.mux.com` (OAuth, "Code Mode") | Add when businesses upload video promos. https://www.mux.com/pricing · https://www.mux.com/docs/integrations/mcp-server |
| **Firebase Storage** | Raw file storage | `*.firebasestorage.app` buckets (us-central1/us-west1/us-east1): 5 GB-months stored + 100 GB/month downloaded free, then Cloud Storage pricing | Firebase MCP has storage tools | Fine for raw originals; no transforms or CDN tricks. https://firebase.google.com/pricing |
| **Cloud Vision SafeSearch** (moderation pick) | Explicit-content scoring | First 1,000 units/mo free; $1.50 per 1,000 up to 5M | n/a | Same GCP project and billing; call it from the upload Function. https://cloud.google.com/vision/pricing |
| **Sightengine** | Image/video/text moderation, AI-image/deepfake detection | Free 2,000 operations/mo (max 500/day). Starter $29/mo (10K ops). Pro $99/mo | n/a | Use if you need text moderation + video. https://sightengine.com/pricing |
| **Ayrshare** | Social posting/analytics/comment moderation API across 14+ networks, multi-tenant "profiles" per business | Premium $149/mo (1 profile). **Launch $299/mo, up to 10 profiles, 28-day free trial.** Business $599/mo (30 profiles), then $8.99 → $1.99 per profile | Action MCP `https://api.ayrshare.com/mcp` (Bearer key); Claude Code plugin `claude plugin marketplace add ayrshare/ayrshare-social-media-api-claude-plugin` | Phase 2 "post your promo everywhere" feature. https://www.ayrshare.com/pricing/ · https://www.ayrshare.com/docs/additional/mcp-claude-code-plugin |
| **Content generation** | Firebase AI Logic (Gemini) for ad copy; Cloudinary Image Generation add-on; Mux Robots "generate contextual text" | AI Logic is billed at Google Cloud/Gemini API rates | Firebase MCP has `ai_init_guide` | Keep generation server-side and keep a human approval step. |

DAM note: Cloudinary Assets (Enterprise) and ImageKit's DAM exist, but a per-org folder + tags in Cloudinary is enough DAM for local businesses.

## 4. B2B SaaS plumbing
| Need | Pick | Cost | Notes / source |
|---|---|---|---|
| **Multi-tenant orgs + roles** | Postgres `orgs`, `memberships(user_id, org_id, role)`; Firebase **custom claims** `{orgId, role}` set by Admin SDK in a Function | $0 | Claims are for access control only, max 1,000 bytes, and propagate on token refresh. https://firebase.google.com/docs/auth/admin/custom-claims |
| **Enterprise SSO (later)** | Firebase Auth with Identity Platform | 50K MAU free; SAML/OIDC 50 MAU free, then GCP pricing | https://firebase.google.com/pricing |
| **Business verification** | Claim via Google Place ID + phone/domain-email check; **Stripe Identity** for edge cases | Stripe Identity: $1.50 per document+selfie, first 50 free (third-party reads of stripe.com, Oct–Nov 2026; Stripe's own page 404'd for me) | https://startwithidentity.com/vendors/identity-verification/stripe-identity/ |
| **CRM (Joe)** | **HubSpot Free** (2 users) or **Attio Free** (3 users, 50K records) | HubSpot Starter $20/seat list. Attio Plus $50/mo (monthly) or $40/mo (annual) | HubSpot remote MCP `https://mcp.hubspot.com` needs an MCP connector created in HubSpot (OAuth + PKCE). It covers CRM, campaigns, marketing email, social. https://developers.hubspot.com/docs/apps/developer-platform/build-apps/integrate-with-the-remote-hubspot-mcp-server. Attio MCP is on every plan; startups get 80% off Pro year 1. https://attio.com/pricing. HubSpot prices are from third-party reads (Nov 2026); HubSpot's page didn't render |
| **Customer support** | **Crisp Free** | Free: 2 seats, chat widget, chat SDKs. Mini $45/mo. Essentials $95/mo (knowledge base, AI chatbot) | https://crisp.chat/en/pricing/ |
| **Email** | Resend (already have) | n/a | Weekly advertiser report email |
| **Admin panel** | **react-admin** (MIT) or **Refine** | Free OSS; react-admin Enterprise from €145/mo | Has a Firebase adapter; for SQL Connect, write a data provider over Functions. https://marmelab.com/react-admin/ · https://refine.dev/ |
| **Internal BI** | Metabase OSS (self-host) on Cloud SQL read replica | Free | Reads `ad_events` for Joe's sales reporting |

## 5. MCP servers / Claude Code plugins (official)
| Server | Install | Auth | Notes |
|---|---|---|---|
| **Stripe** | `claude mcp add --transport http stripe https://mcp.stripe.com/` (or `stripe agent setup` after the Stripe CLI) | OAuth via `/mcp` (preferred); or an **agent API key** as `Bearer ${STRIPE_AGENT_API_KEY}` in `.mcp.json` | ⚠️ Stripe MCP stops accepting full/restricted keys *without the Agent tag* in **Dec 2026**, so migrate by Nov 30, 2026. Refunds need human confirmation in the Dashboard. https://docs.stripe.com/mcp |
| **PostHog** | `npx @posthog/wizard mcp add`, or http `https://mcp.posthog.com/mcp` | OAuth | Free; some AI tools bill PostHog AI credits |
| **Firebase** | `claude plugin marketplace add firebase/firebase-tools` → `claude plugin install firebase@firebase`, or `claude mcp add firebase npx -- -y firebase-tools@latest mcp` | Firebase CLI login | Covers Auth (incl. custom claims), SQL Connect execute/schemas, Functions logs, Storage, rules. Use `--only auth,dataconnect,functions,storage` to cut tools. https://firebase.google.com/docs/ai-assistance/mcp-server |
| **Cloudinary** | `/plugin install cloudinary@claude-plugins-official` | OAuth | Analysis server is OAuth-only |
| **Mux** | http `https://mcp.mux.com` | OAuth (use a read-only token if you want no writes) | Add when you have video |
| **HubSpot** | http `https://mcp.hubspot.com` | OAuth + PKCE via an MCP connector created in HubSpot | Joe's CRM |
| **Attio** | MCP on all plans | Per Attio docs | Alternative CRM |
| **Ayrshare** | Claude Code plugin (see §3) | API key (plugin stores it in the OS credential store) | Paid plan required |
| **Tinybird / Cube / Metabase** | Each has an MCP (per their pricing pages) | n/a | Add when adopted |

Windows note (not verified this run): if `claude mcp list` shows the Firebase stdio server failing on native Windows, try `cmd /c npx ...` as the command.

---

## Copy-paste Claude Code prompt (run from `C:\Users\User\fuse`)

```
You are helping me turn Fuse into ad + analytics + media-management software for local businesses. Read research/business-tools-2026-11-13.md in C:\Users\User\fuse\fuse-research first.

RULES (non-negotiable):
- Ask me before changing any app code, package.json, Firebase config (firebase.json, .firebaserc, dataconnect/, rules), and before any git commit.
- API keys live ONLY in Windows user environment variables. In .mcp.json reference them as ${VAR} (e.g. "Authorization": "Bearer ${STRIPE_AGENT_API_KEY}"). Never write a literal key into any file, command history, or chat. Prefer OAuth wherever the server supports it so no key is needed at all.
- Use Stripe test mode / sandbox and our POC Firebase project only.

STEP 1 – MCP setup (project scope, show me the .mcp.json diff before saving):
- Stripe: http https://mcp.stripe.com (OAuth via /mcp; if I choose a key instead, it must be an Agent-tagged key in ${STRIPE_AGENT_API_KEY}).
- PostHog: http https://mcp.posthog.com/mcp (OAuth).
- Firebase: npx -y firebase-tools@latest mcp --only auth,dataconnect,functions,storage (use "cmd /c" wrapper on Windows if it fails to connect).
- Cloudinary: tell me to run /plugin install cloudinary@claude-plugins-official (OAuth).
- Optional, ask first: Mux (https://mcp.mux.com, OAuth), HubSpot (https://mcp.hubspot.com, needs a HubSpot MCP connector - give me the steps).
Then run `claude mcp list` and report status. If a server needs env vars, give me the PowerShell line using [Environment]::SetEnvironmentVariable("NAME", "<paste>", "User") with a placeholder only.

STEP 2 – Read-only discovery: inspect our SQL Connect schema, Functions, and auth setup (no edits). Summarize what exists for users, plans, places, and events.

STEP 3 – Draft a plan as docs/plans/b2b-ads-v1.md (show me before writing) covering, in this build order:
1. Orgs/memberships/roles tables + Firebase custom claims {orgId, role} set by a Function.
2. Business onboarding: claim a Google Place, verify, Stripe Checkout prepaid budget via webhook.
3. Campaigns/creatives/budgets tables + getSponsoredCards Function (geo/time/category filter, bid x quality rank, daily pacing, frequency cap, "Sponsored" label, 1-in-N deck slots).
4. Server-side ad_events (impression, swipe_right, save, plan_created, redeem) with App Check enforced and per-user dedupe; PostHog for product events.
5. Hourly ad_daily_stats rollup + advertiser dashboard screen in the Expo web build, tenant-scoped by orgId.
6. Cloudinary signed uploads per org + Cloud Vision SafeSearch moderation + react-admin internal approval queue.
7. Later: Stripe Meters/credit grants, Radar, Ayrshare, Tinybird, Kevel/Topsort.
For each step list files to touch, schema changes, env vars needed (names only), POC/Test/Prod differences, and test plan. Stop after the plan and wait for my approval.
```
