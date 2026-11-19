# Event Schema Design

**Team:** Kutateladze & Co.  
**Date:** []  
**Product:** [Working Title: ProofVault - Unified Document & Receipt Vault]  
**Version:** 1.0 (MVP)

---

## Overview

This document defines the **event schema** for our product's analytics instrumentation. Every core user action in the capture → indexing → retrieval flow is tracked with standardized, PII-free properties.

**Purpose:**
- Power our North Star Metric (`Weekly Successful Document Retrievals`) and 3 Leading Indicators
- Measure conversion across the AARRR funnel
- Diagnose where users drop off (e.g., uploading unindexed files vs. failing search)
- Keep MVP instrumentation lean with 8 core events

---

## Event Naming Conventions

**Format:** `object_action` (snake_case)

**Examples:**
- ✅ `document_captured`
- ✅ `document_retrieved`
- ✅ `workspace_invite_sent`
- ❌ `DocumentCaptured` (wrong case)
- ❌ `capture_document` (wrong order)
- ❌ `document_retrieve` (inconsistent verb tense)

**Rules:**
1. Always lowercase with underscores (`snake_case`)
2. Format: `[object]_[action]` (noun first, verb second)
3. Use past tense for completed actions: `_completed`, `_captured`, `_indexed`, `_retrieved`, `_sent`
4. Use `_started` for initiated flows (`user_signup_started`, `user_session_started`)
5. Never embed PII (names, emails, phone numbers, filenames, or raw receipt text) in event names or properties

---

## Standard Properties (All Events)

These properties MUST be included with every event:

| Property Name | Type | Description | Example |
|---------------|------|-------------|---------|
| `event_id` | string (UUID) | Unique identifier for this event | `"550e8400-e29b-41d4-a716-446655440000"` |
| `user_id` | string (UUID) | Anonymized/hashed user identifier | `"usr_9f82c1a4"` |
| `workspace_id` | string (UUID) | Shared household or office vault ID | `"wks_3d19a0b2"` |
| `timestamp` | ISO 8601 datetime | UTC timestamp when event occurred | `"2026-11-18T14:23:45Z"` |
| `session_id` | string (UUID) | Current user session ID | `"ses_7a41b2c9"` |
| `platform` | enum | Client surface | `"web"`, `"mobile_web"` |
| `app_version` | string | Deployed application version | `"0.1.0"` |
| `environment` | enum | Deployment environment | `"production"`, `"staging"`, `"development"` |
| `is_internal_test` | boolean | Flags team QA accounts to exclude from KPIs | `false` |

---

## Core Events (8 Events for MVP)

### Event 1: User Signup Started

**Event Name:** `user_signup_started`

**When it fires:** User opens the signup screen or clicks a workspace invite link to create an account.

**Why we track it:** Measures top-of-funnel Acquisition and signup drop-off.

**Custom Properties:**

| Property Name | Type | Required | Description | Example Values |
|---------------|------|----------|-------------|----------------|
| `signup_method` | enum | Yes | Selected auth provider | `"email"`, `"google"` |
| `entry_source` | enum | Yes | How user arrived | `"direct"`, `"workspace_invite"`, `"beta_referral"` |
| `utm_campaign` | string | No | Outreach batch tag | `"week6_beta"` |

**Example Event Payload:**

    {
      "event_name": "user_signup_started",
      "event_id": "550e8400-e29b-41d4-a716-446655440001",
      "user_id": "anon_83c10f22",
      "workspace_id": "none",
      "timestamp": "2026-11-18T10:15:00Z",
      "session_id": "ses_1102a",
      "platform": "web",
      "app_version": "0.1.0",
      "environment": "production",
      "is_internal_test": false,
      "signup_method": "google",
      "entry_source": "beta_referral",
      "utm_campaign": "week6_beta"
    }

---

### Event 2: User Signup Completed

**Event Name:** `user_signup_completed`

**When it fires:** User finishes registration and their initial vault workspace (or joined shared workspace) is created.

**Why we track it:** Measures Acquisition conversion (`user_signup_completed / user_signup_started`) and segments users by ICP type.

**Custom Properties:**

| Property Name | Type | Required | Description | Example Values |
|---------------|------|----------|-------------|----------------|
| `signup_method` | enum | Yes | Auth method completed | `"email"`, `"google"` |
| `user_segment` | enum | Yes | Self-selected ICP role | `"household"`, `"small_office_admin"`, `"accountant"`, `"business_owner"` |
| `joined_existing_workspace` | boolean | Yes | Whether user joined via invite | `true`, `false` |
| `time_to_complete_seconds` | integer | Yes | Seconds from signup start to finish | `38`, `95` |

**Relationship to other events:**
- Follows: `user_signup_started`
- Conversion metric: `user_signup_completed / user_signup_started` = Signup Completion Rate

---

### Event 3: Document Captured

**Event Name:** `document_captured`

**When it fires:** User uploads a receipt, warranty, invoice, or certificate (via camera, file picker, or share/forward flow) and the file is saved in storage.

**Why we track it:** Tracks raw document ingestion volume across capture channels and feeds the Activation funnel (`capture → index`).

**Custom Properties:**

| Property Name | Type | Required | Description | Example Values |
|---------------|------|----------|-------------|----------------|
| `document_id` | string (UUID) | Yes | Unique ID of captured document | `"doc_4481a9c0"` |
| `capture_channel` | enum | Yes | How document entered vault | `"mobile_camera"`, `"file_upload"`, `"drag_drop"`, `"chat_forward"` |
| `file_type` | enum | Yes | File MIME category | `"image_jpg"`, `"image_png"`, `"pdf"` |
| `contributor_role` | enum | Yes | Role of uploader in workspace | `"owner"`, `"admin"`, `"spouse_family"`, `"staff"` |
| `file_size_kb` | integer | No | File size for latency monitoring | `420`, `1850` |

**Example Event Payload:**

    {
      "event_name": "document_captured",
      "event_id": "550e8400-e29b-41d4-a716-446655440003",
      "user_id": "usr_9f82c1a4",
      "workspace_id": "wks_3d19a0b2",
      "timestamp": "2026-11-18T11:02:14Z",
      "session_id": "ses_1102a",
      "platform": "mobile_web",
      "app_version": "0.1.0",
      "environment": "production",
      "is_internal_test": false,
      "document_id": "doc_4481a9c0",
      "capture_channel": "mobile_camera",
      "file_type": "image_jpg",
      "contributor_role": "owner",
      "file_size_kb": 680
    }

---

### Event 4: Document Indexed

**Event Name:** `document_indexed`

**When it fires:** A captured document completes metadata extraction/tagging and is saved with searchable fields (eliminating `IMG_2041` / `scan_001` dead files).

**Why we track it:** Powers **Leading Indicator #1** (48-hour activation: 3+ indexed documents) and **Leading Indicator #2** (Indexed Metadata Completeness Rate >= 90%).

**Custom Properties:**

| Property Name | Type | Required | Description | Example Values |
|---------------|------|----------|-------------|----------------|
| `document_id` | string (UUID) | Yes | Matching document UUID | `"doc_4481a9c0"` |
| `document_category` | enum | Yes | Type of proof stored | `"warranty"`, `"expense_receipt"`, `"invoice"`, `"utility_bill"`, `"medical"`, `"certificate"`, `"other"` |
| `indexing_method` | enum | Yes | How fields were populated | `"auto_ocr"`, `"auto_ocr_edited"`, `"manual"` |
| `has_vendor` | boolean | Yes | Whether vendor/issuer is present | `true`, `false` |
| `has_document_date` | boolean | Yes | Whether transaction date is present | `true`, `false` |
| `has_amount` | boolean | Yes | Whether monetary amount is present | `true`, `false` |
| `has_expiry_date` | boolean | Yes | Whether warranty/renewal date is set | `true`, `false` |
| `metadata_complete` | boolean | Yes | `true` if `has_vendor && has_document_date` | `true`, `false` |

---

### Event 5: Document Searched

**Event Name:** `document_searched`

**When it fires:** User submits a search query or applies a filter (category, date range, contributor, or active/expiring status) in the vault.

**Why we track it:** Measures search intent and detects zero-result searches (`results_count = 0`) where users fail to find what they saved.

**Custom Properties:**

| Property Name | Type | Required | Description | Example Values |
|---------------|------|----------|-------------|----------------|
| `search_id` | string (UUID) | Yes | Unique ID linking search to retrieval | `"srch_77b12e"` |
| `search_mode` | enum | Yes | Primary lookup method used | `"keyword"`, `"category_filter"`, `"date_range"`, `"expiring_soon"` |
| `query_length_chars` | integer | Yes | Character length (never raw text for privacy) | `0`, `6`, `12` |
| `results_count` | integer | Yes | Number of matching documents returned | `0`, `1`, `4` |

---

### Event 6: Document Retrieved (North Star Metric Event)

**Event Name:** `document_retrieved`

**When it fires:** User opens, downloads, or exports a valid matching document from search/filter results (the exact moment of proof delivery).

**Why we track it:** **Primary North Star Metric event** (`Weekly Successful Document Retrievals`) and measures retrieval speed (`< 15s` target vs. 20 min–2 hrs workaround).

**Custom Properties:**

| Property Name | Type | Required | Description | Example Values |
|---------------|------|----------|-------------|----------------|
| `document_id` | string (UUID) | Yes | Retrieved document UUID | `"doc_4481a9c0"` |
| `search_id` | string (UUID) | No | Linked `search_id` if found via search | `"srch_77b12e"` |
| `document_category` | enum | Yes | Type of proof retrieved | `"warranty"`, `"expense_receipt"`, `"invoice"`, `"medical"`, `"certificate"` |
| `result_action` | enum | Yes | Proof action taken | `"opened"`, `"exported"`, `"shared"` |
| `retrieval_time_seconds` | integer | Yes | Seconds from search/vault open to document action | `4`, `11`, `28` |
| `document_age_days` | integer | Yes | Days since document was originally captured | `0`, `14`, `90` |

**Example Event Payload:**

    {
      "event_name": "document_retrieved",
      "event_id": "550e8400-e29b-41d4-a716-446655440006",
      "user_id": "usr_9f82c1a4",
      "workspace_id": "wks_3d19a0b2",
      "timestamp": "2026-11-25T16:40:12Z",
      "session_id": "ses_8831f",
      "platform": "web",
      "app_version": "0.1.0",
      "environment": "production",
      "is_internal_test": false,
      "document_id": "doc_4481a9c0",
      "search_id": "srch_77b12e",
      "document_category": "warranty",
      "result_action": "opened",
      "retrieval_time_seconds": 9,
      "document_age_days": 7
    }

---

### Event 7: Workspace Invite Sent

**Event Name:** `workspace_invite_sent`

**When it fires:** Workspace owner invites a spouse, family member, driver, staff member, or accountant into their shared vault.

**Why we track it:** Powers **Referral** (AARRR) and **Leading Indicator #3** (`Multi-Contributor Workspace Rate >= 35%`), directly solving Pattern #1's multi-person fragmentation.

**Custom Properties:**

| Property Name | Type | Required | Description | Example Values |
|---------------|------|----------|-------------|----------------|
| `invite_id` | string (UUID) | Yes | Unique invite identifier | `"inv_19d22c"` |
| `invite_role` | enum | Yes | Invited collaborator relationship | `"spouse_family"`, `"staff_contributor"`, `"accountant_viewer"` |
| `invite_channel` | enum | Yes | How invite link was shared | `"copy_link"`, `"whatsapp_share"`, `"email_invite"` |
| `current_member_count` | integer | Yes | Members in workspace before invite | `1`, `2`, `5` |

---

### Event 8: User Session Started

**Event Name:** `user_session_started`

**When it fires:** Authenticated user opens the app and begins a new active session (debounced once per 30-minute session window).

**Why we track it:** Measures Day-7 / Week-2 Retention cohorts and correlates vault size (`workspace_doc_count`) with repeat usage.

**Custom Properties:**

| Property Name | Type | Required | Description | Example Values |
|---------------|------|----------|-------------|----------------|
| `days_since_signup` | integer | Yes | Cohort age in days | `0`, `7`, `14` |
| `days_since_last_visit` | integer | Yes | Gap since previous session | `0`, `3`, `12` |
| `workspace_doc_count` | integer | Yes | Total indexed docs in user's vault | `0`, `5`, `42` |
| `user_segment` | enum | Yes | ICP role | `"household"`, `"small_office_admin"`, `"accountant"`, `"business_owner"` |

---

## Event-to-Metric Mapping

### North Star Metric

**Our NSM:** Weekly Successful Document Retrievals

**Events that power it:**
1. `document_searched` - Tracks intent and provides `search_id` + start timestamp for retrieval latency
2. `document_retrieved` - Primary NSM numerator when `retrieval_time_seconds <= 60`, `result_action IN ('opened', 'exported', 'shared')`, and `is_internal_test = false`
3. `document_indexed` - Prerequisite ensuring documents have searchable metadata before retrieval

**Calculation:**  
`COUNT(event_id WHERE event_name = 'document_retrieved' AND retrieval_time_seconds <= 60 AND result_action IN ('opened', 'exported', 'shared') AND is_internal_test = false AND timestamp WITHIN last_7_days)`  
*(Deduplicated per `user_id` + `document_id` within a 5-minute window to avoid accidental double-click inflation.)*

---

### AARRR Funnel Mapping

| AARRR Stage | Events | Metric Calculation |
|-------------|--------|-------------------|
| **Acquisition** | `user_signup_started`, `user_signup_completed` | Signup Completion Rate = `COUNT(user_signup_completed) / COUNT(user_signup_started)` |
| **Activation** | `document_captured`, `document_indexed`, `document_retrieved` | 48h Activation Rate = `% of signed-up users with >= 3 document_indexed events within 48h` (Leading Indicator #1); Indexing Completeness = `% of document_indexed with metadata_complete = true` (Leading Indicator #2) |
| **Retention** | `document_retrieved`, `user_session_started` | NSM = Weekly `document_retrieved` count; Week-2 Retention = `% of Week-N activated users firing >= 1 user_session_started or document_retrieved in Week N+1` |
| **Referral** | `workspace_invite_sent`, `user_signup_completed` (`joined_existing_workspace = true`) | Multi-Contributor Rate = `% of active workspaces firing >= 1 workspace_invite_sent` (Leading Indicator #3); Invite Acceptance = `joined signups / invites sent` |
| **Revenue** | N/A for MVP | Free beta in Weeks 6–8; paid small-office workspace tier planned for V2 |

---

## Data Types Reference

| Type | Description | Example | Validation Rules |
|------|-------------|---------|------------------|
| `string` | Text value | `"week6_beta"` | Max 255 characters; no PII |
| `integer` | Whole number | `9`, `42`, `0` | Non-negative whole numbers for counts/seconds |
| `boolean` | True/false | `true`, `false` | Lowercase boolean |
| `enum` | Fixed allowed set | `"warranty"`, `"invoice"` | Reject values outside schema enum |
| `ISO 8601 datetime` | UTC timestamp | `"2026-11-18T14:23:45Z"` | Must end in `Z` (UTC) |
| `UUID` | Unique identifier | `"550e8400-e29b-41d4..."` | 36-char UUID or prefixed ID |

---

## Event Volume Estimates

Based on our MVP beta target (**~25 active workspaces / ~35–40 total users** during Weeks 6–8):

| Event Name | Beta Daily Volume (Wk 6–8) | Beta Monthly Volume | Assumptions |
|------------|----------------------------|---------------------|-------------|
| `user_signup_started` | 2–3 / day | ~75 / month | Beta outreach to `icp.md` contacts + referrals |
| `user_signup_completed` | 1–2 / day | ~50 / month | ~67% signup completion |
| `document_captured` | 15–20 / day | ~500 / month | ~25 workspaces uploading ~4–5 receipts/warranties per week |
| `document_indexed` | 14–19 / day | ~475 / month | ~95% capture-to-index completion |
| `document_searched` | 12–15 / day | ~400 / month | Small-office/accountant weekly lookups + household tests |
| `document_retrieved` | 10–12 / day | ~325 / month | ~75/week MVP NSM target (~80% search-to-retrieve) |
| `workspace_invite_sent` | 0.5–1 / day | ~20 / month | ~35–40% of active workspaces inviting spouse or staff |
| `user_session_started` | 10–15 / day | ~375 / month | Repeat sessions across office & household testers |
| **TOTAL** | **~65–85 / day** | **~2,220 / month** | Well within PostHog's 1M events/month free tier |

---

## Implementation Guidelines

### Where to Instrument Events

**Frontend (React Web App):**
- `user_signup_started` (UI open)
- `document_searched` (search/filter submitted)
- `document_retrieved` (document opened/exported/shared + client `retrieval_time_seconds` measurement)
- `user_session_started` (authenticated session init)

**Backend (Express API):**
- `user_signup_completed` (only after user & workspace records persist in DB)
- `document_captured` (only after file upload completes)
- `document_indexed` (only after OCR/manual metadata saves to DB)
- `workspace_invite_sent` (only after invite token is generated)

**Rule of Thumb:** Track UI interaction intent on the frontend and verified state mutations on the backend-never double-fire the same event from both layers.

---

### Event Sending Pattern

- **Asynchronous:** Fire analytics calls without blocking upload, search, or UI rendering (`try/catch` wrapped so analytics outages never break core features).
- **Retry Logic:** PostHog JS & Node SDKs automatically queue and retry failed batches (up to 3 retries with backoff).
- **Batching:** Batch standard UI events every 10–30 seconds; flush immediately on `user_signup_completed`, `document_indexed`, and `document_retrieved`.

---

### Code Example (TypeScript Helper)

    // src/lib/analytics.ts
    import posthog from 'posthog-js';

    export function trackEvent(
      eventName: string,
      properties: Record<string, string | number | boolean>
    ): void {
      try {
        posthog.capture(eventName, {
          ...properties,
          event_id: crypto.randomUUID(),
          timestamp: new Date().toISOString(),
          platform: window.innerWidth < 768 ? 'mobile_web' : 'web',
          app_version: import.meta.env.VITE_APP_VERSION ?? '0.1.0',
          environment: import.meta.env.MODE,
        });
      } catch {
        // Analytics failure must never block document capture or retrieval
      }
    }

    function handleOpenDocument(
      docId: string,
      category: string,
      searchStartedAtMs: number,
      searchId?: string
    ) {
      const retrievalSeconds = Math.round((Date.now() - searchStartedAtMs) / 1000);

      trackEvent('document_retrieved', {
        document_id: docId,
        search_id: searchId ?? 'direct_browse',
        document_category: category,
        result_action: 'opened',
        retrieval_time_seconds: retrievalSeconds,
        is_internal_test: false,
      });
    }

---

## Privacy & Compliance

### PII (Personally Identifiable Information) Rules

**NEVER include in event properties:**
- ❌ User emails, phone numbers, or full names
- ❌ Uploaded filenames (e.g., `"Tamuna_Passport.pdf"`) or raw OCR text
- ❌ Raw search query strings (may contain student names, patient names, or clinic details from `13, 14`)
- ❌ Bank card numbers, invoice taxpayer IDs, or home addresses

**Use instead:**
- ✅ Anonymized UUIDs (`user_id`, `workspace_id`, `document_id`)
- ✅ Boolean flags (`has_vendor`, `has_amount`, `metadata_complete`)
- ✅ Generic category enums (`"warranty"`, `"medical"`, `"expense_receipt"`) and `query_length_chars`

### Data Retention & Consent

- **Storage period:** 12 months for raw MVP beta analytics events; aggregated weekly metrics retained for semester demo reporting.
- **Deletion policy:** Deleting a user account or workspace deletes stored document files/records and removes identifiable user mapping (`posthog.reset()`).
- **Consent:** Clear beta notice at signup explaining anonymized product usage tracking (no third-party ad tracking).

---

## Testing Your Events

### Pre-Launch Checklist

- [ ] Every event fires once (no duplicate frontend + backend firing)
- [ ] All 9 standard properties and required custom properties pass schema validation
- [ ] `document_retrieved` accurately calculates `retrieval_time_seconds`
- [ ] Events appear in PostHog Live Events stream within 60 seconds
- [ ] Zero PII leaks confirmed (no filenames, emails, or search strings in payloads)
- [ ] Team test accounts tagged `is_internal_test: true`

---

## Event Schema Evolution

### Adding & Modifying Events

1. **Adding new events (V2):** Document in this file first, add to `src/lib/analytics.ts`, verify in staging Live Events, then update dashboard queries.
2. **Safe changes:** Adding optional properties or new enum values (e.g., adding `"tax_declaration"` to `document_category`).
3. **Breaking changes:** Renaming keys or changing types requires a `v2` event name until migration completes.

---

## Documentation & References

- North Star Metric: `/02-analytics/north-star-metric.md`
- Analytics Plan: `/02-analytics/analytics-plan.md`
- Problem Statement: `/01-discovery/synthesis/final-problem-statement.md`

---

## Sprint 1 Active Events & Ownership (Lab 6 Reconciliation)

All 5 steps of `03-build/mvp-boundary.md` map 1:1 to existing schema events (zero gaps). Events outside the Sprint 1 journey are deferred to Sprint 2 so every active Sprint 1 event fires in PostHog before the Week 7 gate.

| Step / AARRR | Event | Fired By | Key Properties | Owner | Sprint 1 Status |
|--------------|-------|----------|----------------|-------|-----------------|
| Step 1 · Retention/Entry | `user_session_started` | Frontend (`posthog-js`) | `workspace_id`, `days_since_signup`, `days_since_last_visit`, `workspace_doc_count`, `user_segment` | DK | Active (wire Wk 6) |
| Step 2 · Capture | `document_captured` | Backend (`posthog-node`) | `workspace_id`, `document_id`, `capture_channel`, `file_type`, `contributor_role` | NT | Active (wire Wk 6) |
| Step 3 · Activation | `document_indexed` | Backend (`posthog-node`) | `workspace_id`, `document_id`, `document_category`, `indexing_method`, `has_vendor`, `has_document_date`, `has_amount`, `has_expiry_date`, `metadata_complete` | AJ | Active (wire Wk 6) |
| Step 4 · Lookup | `document_searched` | Frontend (`posthog-js`) | `workspace_id`, `search_id`, `search_mode`, `query_length_chars`, `results_count` | GG | Active (wire Wk 6) |
| Step 5 · North Star | `document_retrieved` | Frontend (`posthog-js`) | `workspace_id`, `document_id`, `search_id`, `document_category`, `result_action`, `retrieval_time_seconds`, `document_age_days` | DK | Active (wire Wk 6) |
| Acquisition (cut) | `user_signup_started` | — | Deferred with full OAuth/onboarding cut in `mvp-boundary.md` | — | Deferred to Sprint 2 |
| Acquisition (cut) | `user_signup_completed` | — | Deferred with full OAuth/onboarding cut in `mvp-boundary.md` | — | Deferred to Sprint 2 |
| Referral (cut) | `workspace_invite_sent` | — | Deferred with multi-user workspace invites cut in `mvp-boundary.md` | — | Deferred to Sprint 2 |


## Team Sign-off

- Giorgi Gogichashvili (`@gogicho`) - Discovery & Metrics - [18.11.2026]
- 
-
-

**Tech Lead approval:**  
Anri Javakhishvili (`@anrj`) -

---

## Change Log

| Date | Version | Changes | Author |
|------|---------|---------|--------|
| [FILL IN: Date] | 1.0 | Initial 8-event MVP schema for ProofVault | Kutateladze & Co. |

---

## Notes & Decisions

We considered tracking `document_ocr_failed` and `document_deleted` as separate events for MVP, but omitted them to keep our schema focused at 8 core events. Instead, OCR/indexing failures are captured cleanly via `document_indexed` with `metadata_complete: false`, which directly powers Leading Indicator #2 without extra instrumentation overhead.