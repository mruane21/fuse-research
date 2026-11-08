# Fuse place data: lowest legal cost with no feature loss (2026-11-08)

Researched 2026-11-08 (America/New_York). Prices in USD per 1,000 billable events unless noted. "Unverified" means I could not confirm it from an official page.

## TL;DR
- Move the swipe cards, Discover tiles and spot pages to **Google Places UI Kit (Essentials)**: $1.00 per 1,000 after 10,000 free a month. The card includes the real photo, star rating and count, price and open-now, and photos aren't billed separately.
- Run search through **Text Search IDs Only**, which is free and unlimited. Filter with `openNow`, `minRating`, `priceLevels` and `rankPreference=DISTANCE`, and render the returned IDs with UI Kit.
- Get lat/lng for distances from **Place Details Essentials** (10,000 free a month). The terms let you cache it server-side for up to 30 days.
- Estimated cost: **$0.027 per DAU per day at 1k DAU ($815 a month)** and **$0.0088 at 10k DAU ($2,650 a month)**. That compares with about $0.38 at 1k DAU under today's pattern ($11.5k a month, which matches your $6–20k).
- The trade-off is design control. UI Kit cards use Google's layout, a fixed field order and required attribution, so a full-bleed Tinder-style photo isn't possible.
- **Correction to the 2026-11-07 doc:** do NOT put Places API content on MapLibre or any other non-Google map. Terms §3.2.3(e) forbids it. UI Kit content is the exception (Service Specific Terms §15.1).

## 1. Google Places UI Kit
| Component | Tier / SKU | What it shows |
|---|---|---|
| Place Details **compact** (iOS `PlaceDetailsCompactView`, Android, JS `gmp-place-details-compact`) | Essentials, **Places UI Kit Query** 0678-4F72-DA7C | Single photo (full-screen lightbox optional), address, rating + count, type, price, accessible entrance, **open-now status** |
| Place Details **full** (`PlaceDetailsView`) | Essentials, UI Kit Query | Photo collage, address, rating + count, type, price, accessibility, open now, **opening hours**, editorial summary, website, phone, reviews tab, plus code, features tab, type highlights, popular times |
| Place Search (nearby or text) | Essentials, UI Kit Query | List of place cards for a query |
| Advanced Place Details / compact | Pro, **Places UI Kit Pro** 42AB-1FB3-B56A | Same content, plus you can hide the name, use custom or removed action buttons, filter reviews and media, and add custom content slots (compact only) |
| Advanced Place List | Pro | Up to 20 places from IDs, horizontal or vertical, `mediaSize` (e.g. `.medium`) |
| Advanced Place Search | Pro | Search list with advanced options |

- **Price (2026-11-08 list):**
  - UI Kit Query: 10,000 free, then $1.00 (to 100k), $0.80, $0.60, $0.30, and $0.08 above 5M.
  - UI Kit Pro: 5,000 free, then $5.00, $4.00, $3.00, $1.50, $0.38.
- **Billing rules:**
  - "Billed per component instantiation, not based on the Places API data included… no additional cost for underlying calls… won't create any additional charges for returning photos."
  - Place Details: billed "each time the `PlaceDetailsQuery` method is called. If you load the same place multiple times, you are billed for each request."
  - Advanced Place List: "billed for each list request, and any change to the list is a new billable event." A list of up to 20 places is one charge.
- **Styling:**
  - Theme tokens on iOS, Android and JS cover colors (surface, onSurface, primary, positive and negative open/closed, and more), fonts (bodySmall, titleSmall, displaySmall and others; any font can be assigned), spacing, border widths and corner radii (container, card, thumbnail, collage).
  - You choose which fields show.
  - Field order is fixed: "rendered in a fixed predefined order."
  - Recommended widths: compact vertical 180–300 px, horizontal 180–500 px, full 250–450 px.
  - **Full-bleed edge-to-edge photo cards: not supported** (no layout or photo-size control is documented).
  - Photo pixel size is not documented.
- **Attribution:**
  - Keep the Google Maps attribution visible. Its color must be one of white, gray or black.
  - Don't remove, alter or obscure it (Service Specific Terms §15.4).
  - Visually separate Google content from yours.
  - Custom buttons that send data to a third party must say so.
  - Custom content in slots must not duplicate ratings, hours or reviews, and must carry no ads.
- **Map:** UI Kit may be used "with or without any map, including a non-Google Map" (§15.1). Lat/lng from it may be cached for 30 days (§15.2).
- **Platforms:**
  - Android and iOS have the same component set: Place Details, Advanced Place Details, Place Search, Advanced Place Search, Advanced Place List, Place Markers and Basic Autocomplete.
  - iOS has it only in the **PlacesSwift** SDK (SwiftUI). The minimum is iOS 16 per release notes, which is unverified.
  - Android uses Fragments and Compose, with a minimum of API 24 (unverified).
  - JS offers web components for the Expo web build.
- **React Native / Expo:** no public wrapper found. Build an Expo native module with ExpoSwiftUI views (iOS) and Compose views (Android), which the Expo Modules API supports. On web, use the JS elements.
- **Real apps:**
  - Statiq (EV charging; Google case study, relaunched Jan 2026).
  - Telenav (press release).
  - Roamly (Google Maps Platform Awards demo).

## 2. Google Places photos
- `photoUri` lifetime: **undocumented**. A Google issue-tracker reply says: "The expiry time is not documented anywhere… I can only advise you to not store the photos at all."
- Photo **names** "cannot be cached" and "can expire" (Place Photos doc, updated 2026-11-03).
- Terms §3.2.3(a) forbid "pre-fetch, index, store, reshare, or rehost Google Maps Content." §3.2.3(b) allows no caching except what the Service Specific Terms list.
  - The Service Specific Terms list only place IDs (indefinitely) and lat/lng (30 days) for Places API and Places UI Kit.
  - **So serving one photoUri to many users, caching photos server-side, or proxying them through a CDN is not permitted.**
  - Persistent on-device image caching (e.g. `expo-image` disk cache) is also not covered by any exception. Treat it as not allowed unless Google says otherwise in writing.
- Official line: Google support "can't provide legal advice or grant exceptions… contact Sales if your use case requires custom licensing" (Places policies page).
- Requirements when you show photos:
  - Show the author attribution, or a larger view that includes it.
  - Link to the photo on Google Maps via `googleMapsUri`.
- UI Kit avoids all of this, because Google renders and fetches the photo.

## 3. Google costs, credits and contracts
- **Free usage per SKU per month:** 10k Essentials, 5k Pro, 1k Enterprise. The $200 credit ended 2025-02-28.
- **Google for Startups Cloud Program:**
  - Scale tier (Pre-Seed to Series A, founded within 3–5 years, under $5k in prior credits): up to $100k in Cloud credits in year 1, plus 20% up to $100k in year 2.
  - Maps credits need a **separate application** after acceptance. A Google Cloud benefits page snippet lists **$600 a month** in Maps credits; the duration isn't stated (unverified). Cloud credits don't automatically cover Maps.
- **Subscriptions:**
  - Starter: $100 a month for 50k calls.
  - Essentials: $275 for 100k calls (includes UI Kit Query and Place Details Essentials).
  - Pro: $1,200 for 250k calls (adds UI Kit Pro and Text Search Pro).
  - Enterprise SKUs (rating, hours, price, photos) are **excluded**.
  - For the UI Kit plan, pay-as-you-go is cheaper than these plans.
- **Discounts:** "at Google's discretion… example qualification might be 10,000,000+ monthly billable events… must be set forth in a signed order form."
- **Resellers:** Google Cloud partners can resell Maps, but no public partner rates exist (unverified).
- **Relaxing caching by contract:** possible only through "custom licensing" with Sales. No public terms exist and nothing guarantees it.

## 4. Alternatives
| Provider | Photos / rating / hours / price | Price per 1,000 | Caching | Notes |
|---|---|---|---|---|
| Foursquare Places API | Premium fields: photos, rating (0–10), hours + open_now, price 1–4, tips, popularity | Pro $15 (500 free); **Premium $18.75**, $15, $11.25 | **PAYG: no caching** except `fsq_place_id` and photo IDs. Enterprise: 24 h on-device only | "Powered by Foursquare" on every screen. Photos endpoint returns up to 50 (default 10) |
| Tripadvisor Terra Discover (legacy Content API sunset 2026-08-31) | Photos, ratings, hours, price level | $0.015 per **location returned** ($15), down to $9 at 5k+; 1,000 free once | Only location ID | Every ID a search returns is billed |
| Yelp Places API | Base: rating, hours, 1 profile photo. Enhanced: 3 photos + price. Premium: 12 photos | 30k calls a month included. $229 / $299 / $643 a month; overage $5.91 / $6.57 / $14.13 (secondary source, unverified) | **24 h**; business ID forever | Trial is 5,000 calls and evaluation only |
| Apple Maps Server API | Place object: name, address, coordinates, IDs. **No photos, ratings or hours** | Free, 25,000 calls a day per team (shared with MapKit JS) | n/a | MapKit `MKMapItemDetailViewController` is a private, unmodifiable modal; its photo and rating content isn't documented (unverified) |
| Mapbox Search Box + Places API (Public Preview) | Hours, price level, rating in metadata. **Photos only from the venue's own website** unless sales grants partner photos | Search Box sessions $3.00 (intro) or $11.50; Places API preview quota 1,000 records a month | "Temporary display only", storage needs an agreement | Not ready for production |
| HERE Discover | Hours; ratings need a supplier contract; no photos documented | About $2.75 after 5k free (snippet, unverified) | Per HERE terms | No photos |
| TomTom Places Search | Legacy POI Details had rating and photo IDs (deprecated); new API not checked | 5k free a month (Discover) | Not checked | Not a fit |

None of them matches Google UI Kit Essentials on price with photos, rating, hours and price all included. Long Island photo spot-checks need API keys, so none were run (unverified).

## 5. Free storable catalog (measured)
**Overture places release 2026-10-21.0**, bbox lon −73.50..−73.30, lat 40.80..40.92 (Huntington area), pulled with the `overturemaps` CLI:
- 10,665 places in total.
- **1,015 food & drink**; 895 at confidence ≥0.7 and not closed:
  - 543 restaurants
  - 238 casual eateries
  - 59 bars and alcohol venues
  - 55 cafés and other non-alcoholic venues
- Huntington Village box: 201.
- Field fill on food & drink: website 85%, phone 91%, socials 63%. **No hours, ratings, price or photos.**
- Sources: Meta 571, BrightQuery 222, Foursquare 173, Microsoft 83.
- Most source timestamps are 2026-10. Releases ship monthly.
- **License: CDLA-Permissive-2.0.** Foursquare-sourced rows are Apache-2.0 and AllThePlaces rows are CC0. Attribution is required.
- Spot-check:
  - Found: Besito, Mac's Steakhouse, Paramount, Cinema Arts Centre, Finnegan's, Neraki, Bistro Cassis, Piccolo, Jonathans Ristorante, Sandbar, Ruvo, Larkfield Lanes (bowling), and 19 sushi matches.
  - Honu Kitchen & Cocktails is flagged `permanently_closed`; that status is unverified.
- **OSM** (Overpass mirror kumi.systems, data as of 2026-05-31), same bbox:
  - Only 157 restaurants, bars, cafés and fast-food places.
  - opening_hours 25%, website 45%, phone 41%.
  - License ODbL (share-alike on derived databases).
- **Foursquare OS Places:**
  - Fields: name, address, coordinates, tel, website, email, socials, categories, date_refreshed and date_closed. **No hours, photos or ratings.**
  - Apache-2.0.
  - Now delivered through the Places Portal as an Iceberg catalog, with an account and token.
  - The PMTiles path in the docs returned 404, so Long Island coverage was not measured (unverified).

## 6. How comparable apps do it
- **Corner:**
  - Its map is built with Mapbox and holds only places users add (275k+). The founders rejected advice to "just scrape the internet."
  - AI search reads public metadata (Google, social, Resy) but returns only user-added places (Business Insider, Sept 2025).
- **Swarm:** Foursquare's own app, running on its own data.
- **Timeleft:** curates partner restaurants itself, using Google-review strength among its criteria.
- **Beli, Partiful, Howbout:** no public writeups on their place data or costs found (unverified).
- **Pattern:** they own a catalog (user-generated or curated), keep only IDs, and pay per render.

## 7. Recommended architecture
1. **Catalog (owned):** Overture places for Nassau and Suffolk, refreshed monthly, in Cloud SQL, with the Google `place_id` stored next to each row (allowed indefinitely). Fill the IDs with free Text Search IDs Only lookups.
2. **Search, any kind:** Text Search **IDs Only** with `locationRestriction`, `openNow`, `minRating`, `priceLevels` and `rankPreference=DISTANCE`. The cost should be $0. That these filters stay on the free SKU is inferred from the field-mask billing rule; check one day of billing to confirm.
3. **Cards, Discover and spot page:**
   - Cards: UI Kit **Place Details compact (vertical)** inside the swipe container.
   - Discover: UI Kit **Place Search** element.
   - Spot page: **Place Details full** (hours, phone, website).
   - One Expo native module (SwiftUI + Compose) plus JS web components.
4. **Distances:** lat/lng from Place Details Essentials, cached at most 30 days server-side. Compute distance on-device.
5. **Maps:** Google Maps SDK on native (free and unlimited) and Dynamic Maps on web (10k free). Keep Google maps wherever raw Places API content appears.
6. **Store only** place IDs, votes and group state. No photos, names, ratings or hours from Google.
7. **Credits and guardrails:** apply for Startups Scale and then Maps credits. Add per-minute quotas, budget alerts, App Check and a server-side daily counter.

### Cost model (30-day month; tiered list prices)
Assumptions per DAU per day: 40 swipe cards, 2 Discover lists, 3 spot pages, 3 searches. Distinct places needing lat/lng each month: 3,000 at 1k DAU and 15,000 at 10k. Web map loads: 5% of DAU.

| Scenario | 1k DAU, per month | $/DAU/day | 10k DAU, per month | $/DAU/day |
|---|---|---|---|---|
| A. Today's pattern, estimated (3 Text Search Enterprise + 59 photos + 3 Details Enterprise) | $11,545 | $0.385 | $56,409 | $0.188 |
| **C. UI Kit Essentials (recommended)** | **$815** | **$0.027** | **$2,650** | **$0.0088** |
| D. UI Kit Pro lists (7 Pro loads a day) | $915 | $0.031 | $5,285 | $0.018 |
| C with $600/mo Maps credit (if granted) | about $215 | about $0.007 | about $2,050 | about $0.007 |

### Risks
- **Design:** you lose full-bleed custom photo cards and control of field order. Google's look and attribution show on every card.
- **Native module:** you build and maintain it yourselves (PlacesSwift needs iOS 16+).
- **Price changes:** Google changed SKUs in March 2025 and could do so again.
- **Vendor lock-in:** this deepens reliance on Google.
- **Terms ambiguity:**
  - §3.2.3(d)(iii) "listings or directory service"; get legal review.
  - Embedding UI Kit cards in animated swipe containers isn't addressed by the docs.
- **Billing inference:** that the IDs Only filters stay free is inferred, not stated.
- **Over-calling:** re-rendering a card re-bills it, so avoid unnecessary remounts.

## Sources
- Pricing list: https://developers.google.com/maps/billing-and-pricing/pricing
- SKU details: https://developers.google.com/maps/billing-and-pricing/sku-details/places
- Subscriptions: https://developers.google.com/maps/billing-and-pricing/subscriptions
- UI Kit iOS: https://developers.google.com/maps/documentation/places/ios-sdk/places-ui-kit-overview · /place-details-ui-kit · /advanced-place-details-ui-kit · /advanced-place-list-ui-kit · /places-ui-kit-custom-styling
- UI Kit Android: https://developers.google.com/maps/documentation/places/android-sdk/places-ui-kit-overview
- UI Kit JS: https://developers.google.com/maps/documentation/javascript/places-ui-kit/place-details
- Place Photos: https://developers.google.com/maps/documentation/places/web-service/place-photos
- Text Search: https://developers.google.com/maps/documentation/places/web-service/text-search
- Places policies: https://developers.google.com/maps/documentation/places/web-service/policies
- Terms of Service (modified 2026-08-26): https://cloud.google.com/maps-platform/terms
- Service Specific Terms (modified 2026-10-28): https://cloud.google.com/maps-platform/terms/maps-service-terms
- Issue tracker: https://issuetracker.google.com/issues/110711934
- Startups: https://developers.google.com/maps/billing-and-pricing/public-programs · https://cloud.google.com/startup/benefits
- Statiq case study: https://mapsplatform.google.com/resources/blog/statiq-improving-user-experience-and-increasing-ev-app-usage-with-places-ui-kit/
- Expo SwiftUI: https://docs.expo.dev/versions/latest/sdk/ui/swift-ui/extending/ · Compose: https://docs.expo.dev/versions/latest/sdk/ui/jetpack-compose/extending
- Foursquare: https://foursquare.com/pricing/ · https://docs.foursquare.com/fsq-developers-places/reference/response-fields · https://docs.foursquare.com/fsq-developers-places/reference/usage-guidelines · https://foursquare.com/legal/terms/apilicenseagreement/ · https://docs.foursquare.com/data-products/docs/places-os-data-schema · https://docs.foursquare.com/data-products/docs/access-fsq-os-places
- Tripadvisor: https://docs.terra.tripadvisor.com/docs/usage-based-pricing · https://docs.terra.tripadvisor.com/docs/caching-policy · https://docs.terra.tripadvisor.com/docs/faq
- Yelp: https://docs.developer.yelp.com/docs/plans · https://docs.developer.yelp.com/docs/places-faq
- Apple: https://developer.apple.com/documentation/applemapsserverapi · https://developer.apple.com/documentation/mapkit/mkmapitemdetailviewcontroller
- Mapbox: https://docs.mapbox.com/api/search/places/ · https://docs.mapbox.com/api/search/search-box/ · https://www.mapbox.com/pricing
- HERE: https://docs.here.com/geocoding-and-search/docs/result-types-place
- Overture: https://docs.overturemaps.org/attribution/ (release 2026-10-21.0)
- Corner: https://www.businessinsider.com/new-social-app-corner-google-maps-gen-z-ai-events-2025-9
- Timeleft: https://timeleft.com/post/the-ultimate-guide-to-timeleft-restaurants/

## Claude Code prompt (read-only audit)
```text
Do a READ-ONLY audit of every Google Places / Maps call in my Fuse project (Windows, PowerShell, repo at C:\Users\User\fuse; Expo/React Native app + Firebase Functions). Do NOT change app code, package.json, or Firebase config, and do NOT commit — ask me first before any of those. Never print or write API keys; keys live only in Windows user environment variables and are referenced as ${VAR}. If you find a literal key in any file, report its path and line only (masked) and stop.

1. Find every call to places.googleapis.com (searchText, searchNearby, places/{id}, /media photos, autocomplete), Maps JS/SDK usage, and any Places client library. For each: file:line, platform (iOS/Android/web/Function), the exact X-Goog-FieldMask, and which SKU it triggers (IDs Only / Essentials / Pro / Enterprise / Enterprise+Atmosphere / Place Details Photos) per https://developers.google.com/maps/documentation/places/web-service/text-search.
2. Estimate calls per user session for: swipe deck, Discover tiles, spot page, search, maps. Show the math.
3. Flag terms risks: any storage/caching of photo names, photoUris, image bytes (including expo-image disk cache or a CDN/Storage proxy), names, ratings, hours, or reviews beyond place IDs (indefinite) and lat/lng (≤30 days); any Places API content shown on a non-Google map (MapLibre/Mapbox/Apple) — Terms §3.2.3(e); missing Google Maps / photo author attribution.
4. Compare against this target design and list the exact changes needed (as a plan, not edits):
   - Search: Text Search with field mask `places.id,nextPageToken` only, using openNow/minRating/priceLevels/rankPreference=DISTANCE filters.
   - Cards/Discover/spot page: Google Places UI Kit (Essentials) — iOS PlacesSwift PlaceDetailsCompactView/PlaceDetailsView, Android Place Details compose/fragment, web gmp-place-details-compact / Place Search element — via one Expo native module.
   - Distances: Place Details Essentials `location` only, cached server-side ≤30 days.
   - Maps: Google Maps SDK native, Google Dynamic Maps on web.
5. Output a markdown table (call site → current SKU → target → est. monthly calls at 1k DAU) and a step-by-step migration plan. Save nothing; print the report in chat. Ask me before creating any file.
```
