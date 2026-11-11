# PaveMind VRM Use Case: Analysis Summary

**Source:** *Pavemind (VRN Use Case) Development Possibilities v1.0* (VisionTrack, Confidential)
**Scope analysed:** §2.1, §2.2, §3 (3.1, 3.2) and §7 (7.1, 7.2). §4, §5 and §6 are not yet covered.
**Status:** Discovery draft. Nothing here is a commitment to build, a delivery date or a cost.

> Items marked **[Assumption]** or **[Validate]** are not confirmed. Figures marked *illustrative* only show how to size the problem and are not measurements.

---

## 1. Problem and value

### Problem
The source brief does not state the problem. It is a list of feasibility questions triggered by one customer conversation about "VRM intelligence … for insurance-related investigations".

**Inferred problem statement [Validate with customers]:**
> When an insurer or claims investigator has an incident and a vehicle registration (VRM), they cannot quickly or defensibly establish whether that vehicle appeared on road-facing camera footage, or where and when. Today this depends on slow, manual, patchy evidence.

### Jobs to be done
- When a claim is disputed or looks suspicious, I want to place a specific vehicle at a specific time and place, so I can settle it faster, reject fraud, or prove liability.

### Value
| Who | Value |
|---|---|
| Insurer / investigator | Faster claim decisions, lower fraud leakage, evidence with time, place and footage |
| VisionTrack | New revenue from footage already held; extends PaveMind's "Elastic Search for video" positioning |
| Footage owner (fleet) | Possible revenue share, but also a privacy and consent exposure |

### Fit with PaveMind
PaveMind already follows Observe → Understand → Explain with API endpoints per stage, a data lake of observations tagged by company, and per-customer cost tracking. VRM search is a **new observation type plus a deterministic search endpoint plus an evidence output**, not a new product.

---

## 2. Potential flow (agreed working design)

### Ingest (external system, asynchronous)
1. An external system calls a **new API endpoint** with video and metadata (timestamp, GPS, device). The company is resolved from the authenticated credential.
2. The video is stored in **blob storage** and a job ID is returned.
3. Pipeline: sample frames → detect vehicles/plates → track → select best crop → OCR → combine reads across frames → validate against plate format.
4. Each sighting is stored: normalised VRM, raw read, confidence, company, video ID, offset, GPS, device, model version.
5. Cost is attributed per customer. Status is available by polling or webhook.

### Search
1. The caller sends VRM, date range, optional location and optional minimum confidence (default 70–80%, to be calibrated).
2. Company is taken from the credential. The search is logged.
3. Exact plus fuzzy matching runs against the sighting index.
4. The response lists sightings with time, offset, location, confidence and plate-crop thumbnail, plus a coverage note.

### Download
1. A request for a sighting returns a time-limited link to a clip around the sighting, not the whole file.

### Decisions already taken
| # | Decision | Implication |
|---|---|---|
| 1 | Processing is triggered by an external caller via a new API | The searchable estate is only what callers send |
| 2 | Return only results above a confidence threshold (70–80%) | Threshold is a filter, not a substitute for human confirmation |
| 3 | Company scoping via companyID | Must come from an authenticated credential, not a trusted header |
| 4 | Footage is stored in blob at ingestion | Clips can be returned. Retention and deletion must be defined |

---

## 3. §2.1 ANPR/OCR capability (analysed)

| Brief question | Answer so far |
|---|---|
| Can the platform read VRMs, or only detect plate regions? | **[Validate]** PaveMind's models (SAM3, DINO via RoboFlow) detect objects and do not read text. Assume a plate-reading step must be added behind the existing provider abstraction. |
| Have ANPR/OCR providers been evaluated? | **[Validate]** Internal knowledge needed. Use a dedicated ANPR/OCR model, not a general LLM, because misread characters are unacceptable as evidence. |
| Existing services extracting vehicle metadata? | **[Validate]** Check whether plate detection for redaction exists, since its crops could feed OCR. |

**Key risks**
- Do not use an LLM as the plate reader. Validate against plate syntax and keep a confidence score.
- Companies scoped by header only is not authentication.
- Confidence is not correctness. A confident misread is still possible.

---

## 4. §2.2 Archive processing

### What is possible
| Brief question | Analysis |
|---|---|
| Can OCR run retrospectively on stored footage? | **Yes for footage already in PaveMind blob storage.** The pipeline can read from blob without re-upload, run as an internal backfill job, and write sightings. **Footage that sits only in VisionTrack's main platform archive is not reachable** unless it is pushed through the new endpoint. **[Validate]** which of these "our archive" in the brief means. |
| Realistic retention periods? | Not knowable from the brief. The window equals the shorter of (a) blob retention in PaveMind and (b) source retention at the caller. **[Validate]** current retention for both. Note that processing is only possible while the footage exists. |
| Significant infrastructure change? | Moderate and additive, not a redesign. Needs: a batch queue with throttling and cost caps, priority lanes so backfill never starves live ingest, idempotent jobs, and **model versioning** so reads can be refreshed when the OCR model changes. |

### Recommendation
- **Do not backfill everything.** Backfill on request, bounded by company, date range and a cost ceiling. A pilot customer's chosen window is enough to prove accuracy and demand.
- Store the **model version** on every sighting so old reads can be superseded.
- Define what happens when footage ages out of blob: sightings must be deleted or flagged, or the search returns links to nothing.

### Open items
1. Does "archive" mean PaveMind blob, VisionTrack's core platform, or both?
2. Current retention periods in each store.
3. Is there a spare compute budget for backfill, or must it share capacity with live ingest?

---

## 5. §3 Retrospective search architecture

### Which approach?
| Option | Assessment |
|---|---|
| Process on demand | No upfront cost, but slow and expensive per search |
| Index during ingestion | Fast search, but OCR cost is paid on all footage before demand is proven |
| **Hybrid** | **Recommended.** |

**Recommended hybrid:** index at ingestion for everything callers send (the agreed flow), plus **on-demand backfill** for older footage already in blob. For the pilot, limit ingest to a scoped set (one company, defined window) so cost and accuracy are learned before scaling.

**Optimisation worth testing:** insurers usually know roughly *when and where* an incident happened. Filtering candidate clips by time and GPS before running OCR can reduce processing sharply, which supports on-demand processing for cases where nothing has been indexed.

### 3.1 Data storage
**Fields to store** (the brief lists five, and more are needed):

| Field | Why |
|---|---|
| Normalised VRM and raw OCR read | Search on the normalised value, keep the raw read for audit |
| Confidence (plate-level) | Threshold filtering |
| Timestamp and **offset within video** | Without the offset, a clip cannot be cut |
| GPS location | Place evidence and location filter |
| Device ID | Brief requirement |
| Company ID | Scoping and access control |
| Video/blob ID | Link to footage |
| Bounding box and track ID | Review and de-duplication |
| Model version and job ID | Reprocessing and audit |
| Plate-crop reference (optional) | Reviewer thumbnail, but largest storage item |

**Storage implication (illustrative, to validate):**
- Sightings are small rows. As an example, 100,000 videos/day × ~20 distinct plates per video ≈ 2 million rows/day. At roughly 0.5 KB/row that is about 1 GB/day (~350 GB/year) before indexes.
- Plate-crop thumbnails would dominate if stored (for example ~15 KB each ≈ 30 GB/day), so decide whether to keep them for all reads or only for matches.
- **Conclusion:** the sighting index is small compared with the video blobs. **Compute (OCR) and video storage are the cost drivers, not the index.**
- **[Validate]** the 100k videos/day design target against real volumes. The brief's §4.1 questions cover this.

### 3.2 Search performance
| Brief question | Analysis |
|---|---|
| How quickly could a search return? | An indexed lookup by (company, VRM, time range) should return in seconds. Fuzzy matching adds cost, so keep it bounded (normalise, then allow limited edit distance on candidates). Clip generation adds more time, so make it a separate, asynchronous download step. |
| Months of footage or only recent data? | Months is practical **for indexed footage**, because it is a database query. The limit is what has been processed and what still exists in blob, not query speed. |

**The latency that matters is time-to-searchable:** footage cannot be found until its ingest job finishes. State this expectation to customers.

**[Assumption / Validate]** The existing data lake and SQL layer may not give low-latency indexed lookups at this scale. Check whether a dedicated indexed store is needed for the sighting table.

### Open items
1. Pilot ingest scope: one company, which window?
2. Is a dedicated indexed store needed, or will the current data layer suffice?
3. Are plate crops stored for all reads or only for matches?

---

## 6. §7 Future capability: live watchlist

### 7.1 How different is it from retrospective search?
| Dimension | Retrospective | Live watchlist |
|---|---|---|
| Latency | Minutes to hours acceptable | Seconds |
| Direction | One VRM queried against stored data | Many watched VRMs matched against a continuous stream |
| Processing | Triggered per upload | Always on, 24/7 |
| Cost of error | A wrong match is reviewed before use | A false alert triggers real-world action |
| Legal and ethical | Evidence review, per case | Ongoing monitoring. Needs lawful basis, DPIA and governance **before design** |
| Operations | Best-effort | Needs SLAs, on-call, alert handling |
| Fit with PaveMind today | Direct (video submitted to API) | **Gap.** PaveMind is built around submitted video, not live streams. |

### Where would detection run?
| Option | Pros | Cons |
|---|---|---|
| **Cloud** | Easy to update models; watchlist stays central | Needs video streamed up (bandwidth, cost); latency; high compute cost |
| **Edge/device** | Low latency; only matches leave the device (privacy benefit) | Depends on device compute; firmware updates; watchlist distribution to devices; accuracy is limited by hardware |
| **Hybrid** | Edge reads plates, cloud matches and alerts | Needs edge capability **and** cloud services |

**My view:** hybrid is the only plausible model at fleet scale: the device reads the plate and sends text plus a crop, and the cloud holds the watchlist and raises the alert. **[Validate]** whether current devices can run plate reading at all. That answer decides the roadmap.

**Stepping stone worth considering: near-live.** Short clips uploaded within minutes, processed by the same pipeline, matched against a watchlist, with alerts within minutes rather than seconds. It reuses the retrospective pipeline and tests demand before committing to true real-time.

### 7.2 Development effort: 6 months or multi-year?
No duration can be given honestly without a technical spike. The sizing below is relative, not a time estimate.

| Component | Reuse from retrospective | Relative effort | Notes |
|---|---|---|---|
| Real-time processing | Little | **Large** | Streaming or low-latency ingest, always-on compute, scaling and back-pressure. Largest unknown if edge is required. |
| Alerting engine | None | Medium | Matching, de-duplication, routing, rate limits, escalation, acknowledgement |
| Watchlist management | None | Medium | Add/remove/expiry, reason and authorisation per entry, bulk import, validation |
| Audit controls | Partial | Medium | Who added what, why, who was alerted, retention. Shares design with retrospective audit. |
| Legal, DPIA, governance | None | Gating item | Not engineering, but can block everything |
| Edge capability (if chosen) | None | Large | Device firmware, OTA updates, hardware limits |

**Opinion:**
- **Near-live, cloud-based** is a plausible incremental project **after** the retrospective pilot proves accuracy and demand.
- **True real-time with edge detection** is a roadmap item, not a short project.
- **Gate:** do not start live watchlist work until the retrospective pilot has validated read accuracy, customer demand, and the legal basis.

**What to do now so live stays possible:** keep the OCR step behind the provider abstraction, store a normalised VRM, store a model version and keep audit logging consistent.

### Open items
1. Do current devices have the compute to read plates on-device?
2. Is "live" meant to be seconds or minutes? Near-live may be enough.
3. Who is permitted to hold a watchlist, and on what lawful basis?

---

## 7. Consolidated risks and open decisions

| Priority | Item | Section |
|---|---|---|
| 1 | Legal basis and data-protection for searching VRMs, and the cross-company access model | §1, §2 |
| 2 | Is the searching company the same as the footage owner? | §2 |
| 3 | Real OCR accuracy by condition (day, night, rain), measured on real footage | §3 |
| 4 | Is human confirmation required before footage is released? | §2 |
| 5 | Meaning of "archive", and retention in each store | §4 |
| 6 | Device capability for on-device plate reading | §6 |
| 7 | Whether a dedicated indexed store is needed | §5 |
| 8 | Real daily volumes and cost per hour of footage (brief §4) | §5 |

## 8. Suggested next steps
1. **Customer discovery** (3–5 insurers): volumes, current process and cost, willingness to pay, what counts as good evidence.
2. **Legal review:** rights over footage, VRM search permissibility, lawful basis, retention.
3. **OCR spike:** run candidate ANPR on 20–50 real clips with known plates. Measure read rate by condition and cost per hour.
4. **Pilot design:** one company, a defined window, confidence threshold as a parameter, human confirmation step.
5. **Then continue analysis** of §4 (scale and cost), §5 (accuracy) and §6 (operational model).
