# Trend Radar: Decisions

Decisions made while grilling [brief.md](brief.md). Newest last.

## D1: Radar v1 runs on Google Trends only
- **Decision:** The first radar scores the attribute vocabulary directly from Google Trends history. Image tagging and daily product snapshots run in the background and become a second signal once about 8-12 weeks of history exist.
- **Why:** Snapshot history starts at zero, so growth, momentum and z-score are meaningless at launch. Trends is the only source with instant history.
- **Consequence:** The attribute vocabulary doubles as the Trends query list. Image analysis moves to phase 2 of the MVP. MVP order in the brief needs reordering.

## D2: v1 vocabulary is about 40 hand-curated terms
- **Decision:** Roughly 8 terms in each of 5 categories (colors, silhouettes, garments, fabrics, aesthetics), kept in a versioned YAML file and curated by hand.
- **Why:** Trends compares at most 5 terms per request and values are relative, so every term needs a shared anchor term. About 40 terms is about 10 anchored requests per refresh, which fits rate limits and keeps charts clean.

## D3: pytrends results are cached in Postgres; the app reads only stored data
- **Decision:** Fetch weekly via pytrends with backoff, store every raw result in Postgres, and serve the app only from stored data. Document the ToS gray area in the README.
- **Why:** pytrends is unofficial and can break or rate-limit. Stored data keeps the demo working and preserves history if it does.

## D4: Weekly granularity everywhere
- **Decision:** Pull about 5 years of weekly Trends data and score on weekly points. The daily snapshot collector (phase 2) still runs daily but aggregates to weekly.
- **Why:** Trends only returns daily data for short windows, and stitching daily windows into a long series is fragile. Weekly is stable and gives enough history for z-scores and seasonality.

## D5: Runway images are a tagger showcase from open datasets, not the trend feed (showcase part superseded by D10)
- **Decision:** The weekly radar stays on Google Trends. Runway images come only from open sources (Fashionpedia, CC-licensed Wikimedia Commons photos) and feed an offline FashionCLIP tagger showcase. Licenses are verified per source before use.
- **Why:** The best runway archives (Vogue Runway, Getty, Launchmetrics) forbid scraping and are copyrighted, which breaks the brief's no-scraping rule. Runway data is also seasonal (about 4 points a year), which doesn't fit weekly scoring.
- **Consequence:** v1 gets a small computer-vision slice (offline tagger plus accuracy against Fashionpedia) without public uploads or inference hosting.

## D6: v1 has no auth; board upload comes after v1, behind login (superseded by D10)
- **Decision:** v1 is public and read-only with no login. The visitor "upload your board" feature is built after v1 and requires sign-in (Supabase Auth).
- **Why:** Login limits upload abuse and inference cost, and v1 has nothing that needs an account. This also answers the brief's auth question.

## D7: Trends-vs-runway comparison is deferred
- **Decision:** v1 is the Trends radar plus the offline tagger showcase. The Trends-vs-runway comparison is not in v1.
- **Why:** It needs a seasonal runway series, and open runway data is too thin to support it reliably.

## D8: Product-snapshot source is deferred to after v1
- **Decision:** No source is chosen for the phase 2 daily snapshot collector. Revisit after v1 ships.
- **Why:** The official routes gate access on partner status or sales. Awin (free feeds, advertiser approval needed) and SerpApi (250 free searches a month, then $25 for 1,000) are the realistic candidates. Data retention terms for both are unverified and must be read before storing anything.
- **Research notes (see also D9 for the stack):** eBay production access is partner-only and mostly resale. Amazon's Creators API needs an approved Associates account. CJ and Rakuten product search is unconfirmed for new publishers. ShopStyle Collective has reportedly shut down. Zalando is EU sellers only. ASOS has no official product API found. Google's Merchant API covers only your own merchant data.

## D9: v1 has no FastAPI and no pgvector (superseded by D10)
- **Decision:** A Python script on a GitHub Actions cron pulls Trends, stores raw results in Supabase Postgres, and computes scores. The Next.js app reads Supabase directly. FastAPI and pgvector arrive with the tagger and upload phases.
- **Why:** Nothing in a Trends-only v1 needs a separate API service or vector search. Leaving them out removes a service to host.

## D10: Core product is board-to-shopping, in one app; the trend radar is a supporting feature
- **Decision:** The main feature is: upload a style board, get the main items detected, and see where to shop each one. It lives in one app with a shared core (vocabulary, tagger, Supabase). The Trends radar stays, as a supporting signal that flags which detected items are rising.
- **Why:** This is the product the user actually wants. Both features need the same vocabulary and tagger, and two apps would double hosting and maintenance for a portfolio piece.
- **Shopping links:** Real shopping sites are possible. Order of effort: (1) search deep-links to retailers per detected item (no API, no approval, link-out only); (2) live SerpApi Google Shopping results at request time, not stored (storage terms unverified); (3) Awin affiliate feeds later, which need approval.
- **Supersedes:** D6 (upload is now core, not after v1), D9 (image inference needs a backend, so FastAPI returns to v1), and the showcase part of D5 (the tagger runs on user boards). D1 stays for the radar's data source. D7 and D8 still hold.

## D11: Boards come in as uploaded images only
- **Decision:** Users upload images (screenshots or saved pictures) directly. Pinterest board links are not supported.
- **Why:** Pinterest forbids scraping boards by URL, and its official API needs app approval. Upload works anywhere with no third-party terms. A Pinterest API connection can be revisited later.

## D12: Board analysis is detect, crop, tag, then analyze
- **Decision:** A garment detector trained on Fashionpedia finds each item in an uploaded photo and crops it. FashionCLIP tags color, fabric and style per crop. A separate analysis step ranks the board's main items by recurrence and prominence across photos, groups near-duplicates (for example by clustering embeddings), and compares them to the trend scores.
- **Why:** FashionCLIP alone labels whole images and cannot tie a color to a garment. Detecting and cropping gives per-item results that map to per-item shopping searches. The ranking step is the data-analysis part of the project.
- **Consequence:** One extra model to host (the detector), so inference hosting is now a v1 question. pgvector may be useful for the embedding clustering.

## D13: Models run locally for now
- **Decision:** The detector and FashionCLIP run on the developer's machine. No inference hosting in v1.
- **Why:** Avoids hosting cost and cold-start problems while the pipeline is built.
- **Consequence:** Reviewers cannot upload to a live app. Login is moot until inference is hosted, so D6's login requirement is deferred to that point.

## D14: No live demo for reviewers yet
- **Decision:** The app is for the developer's own use for now. No hosted demo, sample boards or demo video are planned. Revisit before the project is shown as a portfolio piece.
- **Why:** The user is the only person running it, so the live-demo problem does not exist yet.

## D15: Shopping starts with search deep-links
- **Decision:** For each detected item, the app builds search links to a handful of retailers (for example ASOS, Zara, Nordstrom, Depop). SerpApi live results can be added later.
- **Why:** No API, cost or approval is needed, and it is link-out only. The tradeoff is that links are searches, not exact product matches.

## D16: Build the board analysis first; the trend radar comes after
- **Decision:** The first thing to build is the board analysis: find the most repeated and the most distinct pieces inside a board. Trends data, scoring and the radar follow once that works.
- **Why:** The board feature is what the user wants most, and the in-board analysis is the part that carries the data-analysis work.
- **Consequence:** The MVP order proposed earlier is replaced. The Trends pipeline moves behind the board pipeline.

## D17: FashionCLIP is dropped; "same piece" is detector labels plus color
- **Decision:** The pipeline no longer uses FashionCLIP. The Fashionpedia-trained detector supplies garment category and attributes, and k-means supplies color. Two pieces count as the same when they share a garment category, have a similar color, and have overlapping attributes. A piece that matches nothing else on the board is distinct.
- **Why:** One model instead of two, and every match is explainable and easy to debug. The user asked to drop FashionCLIP.
- **Tradeoff:** Fabric and style terms will be weaker, and visually similar pieces with different labels can be missed. An embedding model (DINOv2 or OpenCLIP, licenses unchecked) can be added later if the misses are bad.
- **Supersedes:** The FashionCLIP parts of D12 (tagging per crop) and D5 (offline tagger showcase). The brief's pipeline line and D2's vocabulary role need revisiting, since the vocabulary can no longer rely on zero-shot matching.

## D18: The detector is trained by the user on Fashionpedia
- **Decision:** The user fine-tunes their own garment detector on Fashionpedia instead of finding a pretrained one.
- **Why:** Full control over the label set and license, and a stronger portfolio story.
- **Consequence:** Training needs compute (free on Google Colab, but slow). The detector architecture must have a license that allows the project's intended use. Ultralytics YOLO is AGPL-3.0, so a permissive alternative (for example a torchvision or DETR-style model) is safer if the app is ever hosted. The architecture is not yet decided. The Fashionpedia image licenses must be checked before redistributing any images.

## D19: Repeated and distinct pieces are scored by photo count and prominence
- **Decision:** A piece's repeat score is the number of different photos that contain a matching piece (D17's matching rule), tie-broken by how much of each photo the piece fills. Distinct pieces are those with no match elsewhere on the board, ranked by prominence. A board-level summary shows how varied or repetitive the board is.
- **Why:** Counting photos rather than detections stops one busy photo from dominating. Prominence keeps a small background item from outranking the main garment. Both are easy to check by eye.

## D20: The detector is a torchvision Mask R-CNN
- **Decision:** Fine-tune a torchvision Mask R-CNN on Fashionpedia.
- **Why:** The license places no strings on hosting or sharing the app later (Ultralytics YOLO is AGPL-3.0). It trains on free Colab. It outputs garment outlines, so the share of a photo each piece fills (the prominence in D19) is measured directly.
- **Tradeoff:** More setup and slower inference than YOLO.

## D21: The trend analysis is out of scope for now and will not be set up
- **Decision:** Nothing about Google Trends, trend scoring, the radar, Supabase or the GitHub Actions cron is built or set up. The project is the board analysis only.
- **Why:** The user wants to focus on the board analysis first and asked not to set up the trend part at all.
- **Supersedes:** D1, D2, D3, D4 and D7 no longer describe active work. D8 (product snapshots) stays deferred. They are kept as history in case the trend part returns.

## D22: Repeats are counted per dimension, plus combos
- **Decision:** Item types, colors, attributes and type+color combinations are counted separately. Each is scored by the number of different photos it appears in, tie-broken by prominence (how much of each photo it fills). The output is a taste profile, for example "white tees and ballet flats, with navy across the board". Anything that appears in only one photo, in any dimension, is distinct. Whole-piece repeats show up as combos.
- **Why:** Taste repeats across different garments, such as a favorite color. Matching whole pieces only (D17) misses that.
- **Supersedes:** The matching rule in D17 and the scoring unit in D19. D19's photo-count and prominence logic still applies, per dimension. D17's choice to drop FashionCLIP stands.

## D23: Findings from the Fashionpedia training annotations (checked, not a choice)
Counted on instances_attributes_train2020.json (45,623 images, 333,401 annotations, 46 categories, 294 attributes). The data was downloaded only to the session scratchpad.
- **No color or fabric attributes.** The 294 attributes are grouped into nickname (153), silhouette (25), neckline type (25), textile finishing (21), textile pattern (18), length (15), opening type (10), non-textile material (10), waistline (7), animal (6) and leather (4). Color must come from k-means only (consistent with D17). Fabric such as cotton or linen is not available.
- **Shoes and bags have no attributes.** All 46,374 shoe annotations and all 7,217 bag annotations have zero attributes, and "shoe" is one class. Ballet flats cannot be told apart from sneakers or boots with Fashionpedia alone.
- **Tops are one class.** "top, t-shirt, sweatshirt" is a single category. T-shirt vs. sweatshirt shows up only through nickname attributes such as "classic (t-shirt)" (5,466 uses against 16,548 tops) and "hoodie".
- **Many frequent attributes are uninformative defaults,** such as "no non-textile material" (65,854), "symmetrical" (60,164), "plain (pattern)" (58,468) and "no waistline". Only 62% of annotations have any attribute.
- **Usable attributes:** 253 of 294 have at least 50 uses. The informative groups are fit/silhouette (regular, tight, loose, a-line, straight), length (mini, midi, maxi), waistline, neckline type, pattern, opening type and some nicknames (jeans, gown, hoodie), and they apply to garments (tops, pants, dresses, skirts, coats), not shoes or bags.
- **Image licenses are mixed.** Per image: CC BY-NC-ND 14,775; CC BY-SA 7,131; CC BY 6,650; CC BY-ND 6,064; CC BY-NC-SA 4,819; CC BY-NC 3,776; Unsplash 1,274; "all rights reserved/not found" 544; Pexels 456; Burst 70; public domain 50; Freestocks 14. Most images are non-commercial or no-derivatives, so the images must not be redistributed. The earlier belief that the dataset is CC BY 4.0 came from a Hugging Face mirror card and does not describe the images. Whether a model trained on them can be used commercially is unresolved.

## D24: The project pivots to fit and returns analytics; the board and trend work is parked
- **Decision:** The portfolio project is now fit and returns analytics on a public clothing fit dataset (tabular data, a prediction model and charts). The board analysis (D10 to D23) and the trend radar (D21) are parked, not deleted.
- **Why:** The detector route has hard problems: training time, no shoe types or color labels in Fashionpedia, and mixed image licenses (D23). Fit analytics needs no GPU or images and has clear data-analysis value for fashion-tech employers.
- **Consequence:** brief.md and the earlier decisions describe the parked project. The brief must be rewritten once the new project's questions are settled.

## D25: The fit project predicts fit for a shopper
- **Decision:** Given a customer's measurements, the item and the size they chose, the model predicts small, fit or large and suggests a better size. Success is measured as prediction accuracy on held-out data. A demo lets you enter your measurements.
- **Why:** It gives a clear, measurable result and a demo, and it is the strongest data-analysis story for the chosen direction (D24).

## D26: The dataset is ModCloth
- **Decision:** Train and evaluate on the ModCloth clothing fit dataset (about 83,000 transactions, 5,000 products, per reports). RentTheRunway is not used for now.
- **Why:** The user chose it. It has more varied items than RentTheRunway, at the cost of less data for the model.
- **To check:** Field list, missing values and license, once the file is downloaded from Kaggle (needs a free Kaggle account). The counts come from papers, not from the files.

## D27: Success means beating simple baselines, reported per class
- **Decision:** Compare the model against "always predict fit" and a simple size rule. Report how well each class (small, fit, large) is caught, not just overall accuracy.
- **Why:** If most reviews say "fit", a model that always answers "fit" looks accurate without being useful. Per-class results show whether the model catches the misfits that cause returns.
- **To check:** The real class balance in the downloaded file. Whether "fit" dominates is assumed, not yet seen.

## D28: The demo is a try-it form plus a findings page
- **Decision:** A form takes measurements, an item and a size, and returns likely too small / fits / too large plus a better size. A separate findings page shows the main results, the baseline comparison, and charts of which categories and sizes fail most.
- **Why:** It covers both the prediction and the analysis, and it is what a hiring manager will open.

## D29: The app is Next.js plus FastAPI
- **Decision:** A Next.js website for the form and findings page, calling a FastAPI Python service that serves the model.
- **Why:** The user chose a polished custom site and a stronger software story over Streamlit's speed.
- **Tradeoff:** Two apps to build and host. Streamlit would have been faster for a beginner.
- **To resolve:** The nested trend-forecast folder is a Vite app, not Next.js. It will be replaced by a Next.js app in web/, or the plan changes; the user has not said which.

## D30: The fit project runs locally for now
- **Decision:** No deployment yet. Everything runs on the user's laptop, and hosting is decided at the end.
- **Why:** Less setup while the model and analysis are built. A shareable live link is deferred.

## D31: The Vite folder is left alone until the web step (default, not confirmed)
- **Decision:** The nested Vite app is not touched now. When the web step starts, a Next.js app is created in web/ and the Vite folder is removed or ignored.
- **Why:** It is reversible and does not block the data work. The user has not said what they want done with it.

## D32: Findings from the ModCloth file (checked, not a choice)
Counted on data/modcloth_final_data.json (one JSON record per line). The file is git-ignored.
- **Size:** 82,790 rows, 47,958 users and 1,378 items. The papers reported 5,012 products; the file has 1,378 distinct item ids. Median 3 rows per item, and only 393 items have 10 or more rows. 16,088 users appear more than once (up to 27 rows). 377 rows are exact duplicates.
- **Class balance:** fit 68.6%, large 15.8%, small 15.7%. The assumption in D27 holds: "always predict fit" scores 68.6% accuracy, so accuracy alone is misleading.
- **Missing measurements:** height 99% present, bra size 93%, cup size 92%, hips 68%, shoe size 34%, bust 14%, waist 3%. Waist and bust are too sparse to use. Height, bra size, cup size and hips are usable.
- **Format problems:** 2,898 heights are not "XftYin" (for example "5ft"). Shoe size has values up to 38 (probably a different sizing scale). One bust value is not numeric. Catalog size has 29 distinct values from 0 to 38, which probably mix sizing scales, so size means different things across items.
- **Leakage risk:** The columns length (too long or short), quality (rating), review_text and review_summary are reported after wearing the item. They would not be known when a shopper asks "will this fit?". They must not be features. Length alone is strongly tied to fit (for example "very short" items were "small" 59% of the time), so using it would inflate results.
- **Weak category labels:** "new" and "sale" (about 29% of rows) are merchandising labels, not garment types. Garment types are dresses, tops, bottoms, outerwear and wedding. Fit rates differ little by category (63% to 80% "fit"), so a category-only findings chart will look thin.
- **Personal data:** Every row has a user_name (real first names or usernames) and a user_id, and 92% have free-text reviews. These must be dropped before anything is shared, shown in the app or published.
- **Limit on "suggest a better size":** The file records only the size bought and how it fit. It never shows what size would have fit, so a size suggestion can only be inferred from the model and must be presented as an estimate.

## D33: Train and test are split by shopper
- **Decision:** Each user_id appears on only one side of the train/test split. The same shopper never shows up in both.
- **Why:** 16,088 users have more than one row. If one user is on both sides, the model can memorize their quirks and the test score is inflated. The demo is for a new shopper, so the test should use shoppers the model has never seen.
- **Tradeoff:** Items are still shared across the split, so the model can learn item quirks such as "this dress runs small". The score does not show how well it guesses for a brand-new product.

## D34: "new" and "sale" rows stay in the model but are left out of category charts
- **Decision:** The model trains on all rows, including "new" and "sale". The "which categories fail most" charts show only real clothing types: dresses, tops, bottoms, outerwear and wedding.
- **Why:** "new" and "sale" are shop sections that mix all clothing types, so a bar for them would mean little. Dropping them would lose about 29% of the data.

## D35: The dataset license is CC BY 4.0 (reported by the user, attribution required)
- **Decision:** Treat the ModCloth data as CC BY 4.0. The README must credit the source (Misra, Wan and McAuley, RecSys 2018, and the Kaggle listing), link to the license, and state what was changed (cleaning, dropped columns). Nothing may suggest the authors endorse the project.
- **Source of this fact:** The user opened the CC BY 4.0 page after checking the Kaggle listing. The Kaggle page itself was not seen in this session, so the exact wording on that listing is unverified.
- **Limits:** CC BY does not override privacy. user_name, user_id and review text are still dropped before anything is shown or shared, and the raw file is not republished. The data was originally collected from ModCloth's website, so the uploader's license may not cover the retailer's rights.

## Still open
- None. The planning questions are settled.
