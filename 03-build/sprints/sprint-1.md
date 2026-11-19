# Sprint 1 Plan

**Definition of Done:** A story is done when it is merged to `main`, deployed at the public URL, walked on a phone by a teammate who did not build it, and its events show in the PostHog live view.

## Team Hours Budget 

| Member | Free Hours (before Week 7 lab) | Assigned Hours |
|--------|--------------------------------|----------------|
| DK | 6h | 6h |
| AJ | 8h | 8h |
| GG | 4h | 4h |
| NT | 4h | 4h |
| **Total** | **22h** | **22h** |

## Sprint 1 Stories

| Story | Owner | Reviewer | Hours | Event it must fire |
|-------|-------|----------|-------|--------------------|
| S6 Deploy Vercel, Express, Supabase, PostHog (first evening) | AJ | GG | 2h | all 5 journey events in PostHog Live Events |
| S1 Open/create ProofVault workspace on phone or web | DK | GG | 3h | `user_session_started` |
| S2 Capture or upload receipt/warranty photo or PDF | NT | DK | 4h | `document_captured` |
| S3 Extract and save searchable document metadata | AJ | NT | 6h | `document_indexed` |
| S4 Search and filter stored documents | GG | AJ | 4h | `document_searched` |
| S5 Open or export matching proof in under 60 seconds (North Star) | DK | AJ | 3h | `document_retrieved` |

---

### S6 · DEPLOY STACK & POSTHOG PIPELINE · AJ · 2H (Reviewer: GG)
*As a team shipping to real testers before the break, I deploy a live frontend, API, database, and PostHog pipeline on evening one so every story lands at a public URL.*

```text
Given an empty production deployment on Day 1
When the starter app and Express API are pushed to main
Then the public Vercel URL loads on a phone browser without console or CORS errors
And a test health ping reaches the Express API and Supabase Postgres
And one test event appears in PostHog Live Events with is_internal_test: true
```
**Demo:** Open the public Vercel URL on a phone, trigger the test action, and show the event arriving in PostHog Live Events within 60 seconds.

---

### S1 · WORKSPACE ENTRY · DK · 3H (Reviewer: GG)
*As a household keeper or small-company accountant, I open my ProofVault workspace in one step so I can immediately store or look up a document without profile setup.*

```text
Given the public ProofVault URL on a phone browser
When I enter a workspace name or open an existing workspace link
Then the vault opens directly to the Capture and Search view without pinch-zoom
And user_session_started fires once with user_segment, workspace_doc_count, days_since_signup, and days_since_last_visit
And reloading the page within 30 minutes does not duplicate user_session_started
```
**Demo:** Open the public URL on a phone, create/enter a workspace in under 15 seconds, and show the single `user_session_started` event in PostHog.

---

### S2 · CAPTURE DOCUMENT · NT · 4H (Reviewer: DK)
*As a keeper holding a fresh receipt or warranty slip, I snap a photo or upload a PDF into my vault so it cannot fade or get lost in a drawer or WhatsApp chat.*

```text
Given an open ProofVault workspace on a phone
When I tap Capture, take a photo (or select a JPG/PNG/PDF), and submit
Then the file persists in Supabase Storage and appears in the vault list within 5 seconds
And document_captured fires from the backend with document_id, capture_channel, file_type, and contributor_role
And no filename or PII appears anywhere in the PostHog event payload
```
**Demo:** Snap a real paper receipt on a phone, upload it, reload the page to prove it persisted, and show `document_captured` in PostHog.

---

### S3 · INDEX METADATA · AJ · 6H (Reviewer: NT)
*As a keeper who hates renaming files like IMG_2041 or scan_001, I have the system extract and let me confirm vendor, date, category, and amount so every saved file is searchable.*

```text
Given a newly uploaded receipt or warranty image
When the backend finishes extraction and I tap Save
Then the document record stores vendor, document_date, category, and amount in Postgres
And document_indexed fires from the backend with document_id, document_category, indexing_method, has_vendor, has_document_date, has_amount, has_expiry_date, and metadata_complete
And if extraction misses a field, I can edit it before saving (indexing_method: "auto_ocr_edited")
```
**Demo:** Upload a receipt photo, watch vendor/date/category populate (edit one field if needed), save, and show the updated record plus `document_indexed` in PostHog.

---

### S4 · SEARCH & FILTER · GG · 4H (Reviewer: AJ)
*As a keeper facing an urgent boiler breakdown or tax deadline, I search by vendor/keyword or filter by category so I do not waste hours digging through folders.*

```text
Given a workspace with at least 3 indexed documents
When I type a vendor keyword (e.g., "Bosch" or "PSP") or select a category filter
Then matching documents appear in under 2 seconds, and non-matching documents are hidden
And document_searched fires with search_id, search_mode, query_length_chars, and results_count
And raw search query text is never sent in the PostHog payload
```
**Demo:** On a phone with 3+ docs in the vault, search by vendor and filter by category, showing instant filtered results and `document_searched` (`query_length_chars` only, no text) in PostHog.

---

### S5 · RETRIEVE PROOF (NORTH STAR) · DK · 3H (Reviewer: AJ)
*As a keeper who needs to prove a payment or warranty right now, I open or download the clear original document in under 60 seconds so I avoid paying twice or getting fined.*

```text
Given search or filter results on a phone
When I tap a matching document to view or download the original image/PDF
Then the legible full-resolution document opens on screen within 3 seconds
And document_retrieved fires with document_id, search_id, document_category, result_action, retrieval_time_seconds, and document_age_days
And retrieval_time_seconds measures seconds from search/vault open to opening the document
```
**Demo:** From the vault home screen on a phone, search for a saved receipt and open the readable original in under 15 seconds, then show `document_retrieved` in PostHog.

---

**Slip rule, written now:** If **S3** (automated Vision/OCR extraction) is not working reliably by the evening before the Week 7 lab, AI extraction moves to Sprint 2 and **S3** ships with a fast 3-field manual tag form (vendor, date, category) on upload (`indexing_method: "manual"`). **S4** (`document_searched`) and **S5** (`document_retrieved`) stay in Sprint 1 no matter what, because without searching and opening legible proof in under 60 seconds the value hypothesis cannot be tested.