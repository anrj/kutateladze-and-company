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
---

## Sprint Changes Log

*Original plan above is kept as written. This section records what changed during the sprint and why.*

### 24.11.2026 · Scope pivot: Georgian document Q&A (Nuskha)

**Decision:** The product moved from a receipt/warranty photo vault with OCR field extraction to **search and question-answering over Georgian office documents** (contracts, invoices, HR orders, spreadsheets), with every answer citing the exact file and page. Reason: the value hypothesis is the same (find the right document in seconds instead of digging through folders), but the Georgian-specific problems (pre-Unicode text, translit queries, word endings) are a stronger differentiator than generic receipt OCR. See `03-build/architecture/architecture.md` and `DECISIONS.md`.

**Stack change:** React + Express + Supabase → Solid 2 + Vite (web), Python FastAPI (engine), Neon Postgres with pgvector/pg_trgm, Gemini API, Cloudflare R2 for files. Event schema updated to v1.1 (4 events), see `03-build/analytics/event-schema.md`.

### Story changes

| Story | Original | Changed to | Event (schema v1.1) | Status |
|-------|----------|-----------|---------------------|--------|
| S6 Deploy & PostHog | Vercel + Express + Supabase + PostHog | Vercel/Cloudflare Pages (web) + Render (engine) + Neon + PostHog EU | all 4 journey events | 🟡 Neon connected and PostHog project created; app runs locally. **Public deploy not done yet.** |
| S1 Workspace entry | Create/open a named workspace; `user_session_started` | **Cut.** No accounts or workspaces in the MVP: an anonymous per-browser ID scopes documents and is the PostHog `distinct_id`. App opens straight to upload + ask. | — (event dropped) | ✅ Not needed |
| S2 Capture | Photo or PDF upload of receipts | Upload of office documents: PDF, Word/Excel/PowerPoint (incl. 97–2003 .doc/.xls/.ppt), ODT, RTF, CSV, HTML, TXT. **Photos/scans not supported yet** (no OCR). | `document_captured` | 🟡 Works locally; event rename pending |
| S3 Index metadata | Vision/OCR extracts vendor, date, category, amount; manual edit fallback | **Slip rule applied:** OCR field extraction moved to Sprint 2. Instead: text extraction per page/slide/sheet, Georgian normalization (incl. automatic conversion of pre-Unicode keyboard-layout text), chunking and embeddings for search. No manual tag form needed: documents are searchable by their full text. | `document_indexed` (status, page/chunk counts, georgian_ratio, legacy_text_fixed) | 🟡 Works locally; event rename pending |
| S4 Search & filter | Keyword search + category filter | Natural-language question (Georgian or Latin-typed Georgian) → hybrid search (meaning + keywords) → answer citing file and page. Category filter cut. | `document_searched` (search_id, query_length_chars, results_count, answer_mode) | 🟡 Works locally; event rename pending |
| S5 Retrieve proof ⭐ | Open/download original in < 60 s | Click a cited source → original opens at the cited page; the open is stored in `source_opens` (North Star rows for the reconcile check). | `document_retrieved` (document_id, search_id, page, citation_rank, retrieval_time_seconds) | 🟡 Works locally; `retrieval_time_seconds` + rename pending |

### Acceptance criteria updates

- **S2:** "Given an open ProofVault workspace" → "Given the public app URL". "Take a photo" removed until OCR ships. `capture_channel` and `contributor_role` removed from the event (no camera flow, no workspace roles).
- **S3:** Vendor/date/category/amount fields replaced by: document shows **Ready** with page count and Georgian share; legacy-text badge when pre-Unicode Georgian was converted; failures show the reason (e.g. scanned PDF without text).
- **S4:** "type a vendor keyword or select a category" → "ask a question in Georgian or translit"; "matching documents appear in under 2 seconds" → "an answer with at least one cited source appears" (Gemini answer time varies; if the AI is busy, the best matching passage is shown instead). Raw query text still never sent to PostHog.
- **S5:** `document_category`, `result_action`, `document_age_days` removed from the event; `page` and `citation_rank` added.

### Done so far (outside the story table)

- End-to-end tests for the full journey (upload → index → ask → cited answer → open source) on Postgres: 36 passing.
- Georgian handling: Mtavruli folding, translit queries, pre-Unicode text detection by content, trigram matching for word endings.
- Resilience: retries and fallback Gemini models; the journey still completes if the AI is overloaded.
- Per-user rate limit on questions (cost protection for the public URL).

### Remaining before the Week 7 gate

1. Rename app events to schema v1.1 and add `search_id` / `retrieval_time_seconds` / indexing `status`.
2. Deploy: engine on Render (`render.yaml`), files on R2, web on Vercel/Cloudflare Pages; `APP_ENV=production`.
3. Check events in PostHog Activity from production; fill in the **Live** column of the event schema.
4. Reconcile check: `document_retrieved` count in PostHog = `source_opens` rows in Neon.
5. PostHog dashboard (North Star trend, funnel, retention) with the internal-user filter on; shared link in README.
6. A teammate who didn't build it walks the journey on a phone with Wi-Fi off (Definition of Done).
7. Log AI-assisted work in `docs/ai-usage-log.md`; add the stack change to `DECISIONS.md`.

### Moved to Sprint 2

- OCR for photos and scanned PDFs (Gemini vision), restoring the camera capture flow
- Workspaces, accounts, invites (`user_signup_*`, `workspace_invite_sent`)
- Category/date filters
- Reranking of search results