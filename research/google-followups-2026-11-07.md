# Fuse: Google follow-ups — November 7, 2026 (ET)

## 1. Photo metadata, free linking, and actual image charges

**`places.photos` in Text Search (New) triggers Text Search Pro**, not Essentials and not Enterprise (unless another requested field raises the request to Enterprise).

Official Text Search documentation says “The following fields trigger the Text Search Pro SKU” and explicitly includes `places.photos`:
https://developers.google.com/maps/documentation/places/web-service/text-search

The official cross-API field table independently confirms:
> Photos | `photos` | Place Details Essentials (IDs Only) | Text Search Pro | Nearby Search Pro
https://developers.google.com/maps/documentation/places/web-service/data-fields

**Important cheaper route:** Text Search with `X-Goog-FieldMask: places.id` to link an independently sourced venue; then Place Details with `X-Goog-FieldMask: id,photos` when photo metadata is needed. Both masks remain in their respective free IDs-only SKUs. In Place Details the field is `id`, not `places.id`. Adding displayName, address, rating, etc. can change billing; do not silently add them.

Photo metadata contains resource names, dimensions and attribution, **not downloaded images**. Official docs:
> “Using Place Photos (New), you can then access the referenced photos and resize the image to the optimal size for your application.”
https://developers.google.com/maps/documentation/places/web-service/place-photos

Actual image/media retrieval bills separately as **Places API Place Details Photos**: first 1,000 monthly events free, then $7/1,000 in the first paid band. Text Search Pro: first 5,000 free, then $32/1,000. Prices USD; aggregate usage across projects on the billing account.

**Yes: Text Search Essentials (IDs Only) is currently free with an “Unlimited” free usage cap.** The live official pricing table lists:
> “Places API Text Search Essentials (IDs Only) … Unlimited”
> “Places API Place Details Essentials (IDs Only) … Unlimited”
https://developers.google.com/maps/billing-and-pricing/pricing

“Unlimited” means no paid usage threshold for those SKUs, **not exemption from rate limits, billing/key requirements or terms**. IDs-only Place Details refresh is likewise free. `places.name`/`name` means a resource name, not the business display name. IDs-only matching cannot inspect Google's address/location/display name for ambiguity without requesting other fields; measure wrong-branch matches before automating acceptance.

Official billing rule:
> “You are then billed at the highest SKU applicable to your request.”
https://developers.google.com/maps/documentation/places/web-service/usage-and-billing

### Correction to the earlier cost report
Do **not** implement “download Google photos once and keep in Cloud Storage,” or a permanent Google-derived venue catalog, on the assumption weekly refreshing makes it compliant. Official policies say:
> “Cache or store Google Maps Content only as expressly permitted…”
> “You can therefore store place ID values indefinitely.”
https://developers.google.com/maps/documentation/places/web-service/policies

Photo docs explicitly warn:
> “You cannot cache a photo name. Also, the name can expire.”
https://developers.google.com/maps/documentation/places/web-service/place-photos

Keep independently licensed venue fields/photos separate from Google data. Fetch fresh Google content on interaction, respect attribution and map-display restrictions, and obtain legal review for the linking/enrichment architecture. Free pricing is not permission for bulk extraction.

## 2. Daily versus per-minute quotas: confirmed facts and limitation

**Confirmed specifically for Places API (New): independent per-method, per-project per-minute quotas.** Official wording:
> “The rate limit per minute is per API method per project. In other words, each API method has a separate quota.”
https://developers.google.com/maps/documentation/places/web-service/usage-and-billing

**Daily capping is advertised by Google's general Maps documentation**, which says:
> “Google Maps Platform provides ways to set daily request limits and set maximum daily billable limits.”
https://developers.google.com/maps/faq#usage_quotacap

The Places reporting overview lists requests “Per day”, “Per minute”, and “Per user per minute (where available)”:
https://developers.google.com/maps/documentation/places/web-service/report-monitor

**But these public pages do not enumerate exact daily quota rows for Text Search (New) and Place Photos (New). I could not verify that either has an editable daily row, nor its exact display name, from official public docs. Therefore do not promise a `requests per day` row exists for these two methods.** Generic daily-cap guidance alone is insufficient to prove that.

Exact documented navigation:
1. Select the correct POC/Test/Prod project in Google Cloud console.
2. **APIs & Services → Enabled APIs & services → Places API (New) → Quotas** (may be labeled **Quotas & System Limits**).
3. Filter the table for the relevant method: Text Search (`SearchText`) and photo-media retrieval (`GetPhotoMedia`). These are method identifiers, **not verified console quota display names**.
4. Inspect the listed units. Edit an available quota: checkbox → **Edit quotas** → new value → **Submit request**.
5. If a daily row exists, lower it; if only per-minute rows exist, there is no verified console-only daily cap for that method. Enforce an atomic server-side daily/monthly counter before paid requests, shared across all Function instances. Budgets are alerts, not a hard stop. Do not substitute 1 request/minute for 1 request/day.

Alternative officially documented path: **Google Maps Platform → Quotas → select API → checkbox → Edit → Submit request**.
https://developers.google.com/maps/documentation/places/web-service/usage-and-billing

Generic console instructions and caveat:
> “Quota enforcement has a latency period after a threshold is surpassed, so include a safety buffer…”
https://cloud.google.com/apis/docs/capping-api-usage

**Outstanding verification:** obtain a screenshot/export of this project's Places API (New) quota table to settle exact row names and daily availability. No cloud resources were changed.

## 3. Foursquare OS Places + OSM coverage on Long Island

**No defensible Nassau/Suffolk completeness percentage versus Google is available from this research.** No Google API calls were made, and Google does not expose an exhaustive local venue count through a single Text Search: official docs say a maximum of 60 results across all pages. A result count is not recall or a census.
https://developers.google.com/maps/documentation/places/web-service/text-search

### Foursquare access is now gated
The official dataset page currently states:
> “Foursquare OS Places is now a gated dataset on Hugging Face.”
It lists an October 15, 2026 release, Parquet access examples and Apache 2.0 licensing:
https://huggingface.co/datasets/foursquare/fsq-os-places

I did not bypass the gate or download a global dataset. A local Huntington count was **not measured**. The page documents filtering `date_closed IS NULL` and checking `date_refreshed`; use those filters in the trial rather than counting every historical record. Free/open licensing does not mean anonymous immediate download access.

### OSM measurement attempt
Attempted a public Overpass query using New York county administrative areas named Nassau County and Suffolk County and `amenity=restaurant|bar|pub|cafe|fast_food`, retrieving nodes/ways/relations with tags and centers. Planned downtown sample bbox (south,west,north,east): **40.855,-73.440,40.885,-73.410** — a deliberately defined downtown sample, not an official village boundary or the whole Town of Huntington.

The Python HTTPS request failed certificate verification; a public WebFetch fallback returned HTTP 406. **No counts or metadata percentages were obtained.** Do not report zeros, or infer absence of venues, from these failures. Public endpoint attempted: https://overpass-api.de/api/interpreter

### Verdict and half-day trial
**Promising as a low-cost catalog seed, not yet proven to replace Google for local discovery or live hours/photos.** Evaluate the combined deduplicated union, not the sum of raw counts. OSM tag completeness is not venue recall; a phone/hours tag being present does not establish accuracy.

1. Start with 50–100 independently verified currently operating Huntington restaurants/bars/cafes, balanced by chains/independents/new openings. Build the reference set from merchant sites/local business directories, not only Google. Sample Nassau and eastern Suffolk separately before generalizing island-wide.
2. Fetch OSM and, after authorized access, current FSQ records for identical polygons/categories. Record source release date. Deduplicate by normalized name/address, coordinates and manual review; eliminate closed venues and node/way duplicates.
3. Report per source and combined: raw records, unique active venues, benchmark recall, false positives/closures, duplicates, and percentage with **verified** address/coordinates, website, phone and hours. Count usable licensed photos separately.
4. For each open-data venue, run Google IDs-only name+address lookup; measure no-match, ambiguous match and wrong-branch rate. Do not assume the first ID is correct. Optional paid-field verification must be explicitly bounded and approved first.
5. Replay a representative swipe/detail session; count paid SKU calls and photos per user. Test atomic daily/monthly caps under concurrent Function instances and confirm fallback UI when caps are exhausted.

Decision: use the open-data union plus merchant-supplied corrections/photos as Fuse's owned catalog if recall and freshness meet an agreed threshold; keep Google IDs and selective, on-demand Google enrichment. Do not promise Google-equivalent coverage or hours before the trial.
