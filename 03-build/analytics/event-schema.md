# Event Schema Design

**Team:** Kutateladze & Co.
**Date:** 24.11.2026
**Product:** Nuskha (ნუსხა): Georgian document search & Q&A
**Version:** 1.2 (MVP, Sprint 1)

---

## Overview

This document defines the analytics events for the MVP journey: **upload → index → ask → open the cited source**. Each event maps to one step of `03-build/mvp-boundary.md`. Properties contain no PII.

**Purpose:**
- Power the North Star Metric (`Weekly Successful Document Retrievals`) and 3 Leading Indicators
- Show where users drop off: upload fails to index, search finds nothing, answer found but not opened
- Keep instrumentation lean: **4 events**, no properties that PostHog already records

---

## Event Naming Conventions

**Format:** `object_action` (snake_case, past tense)

- ✅ `document_captured`, `document_retrieved`
- ❌ `DocumentCaptured` (wrong case), `capture_document` (wrong order), `document_retrieve` (wrong tense)

**Rules:**
1. Lowercase with underscores (`snake_case`)
2. Noun first, verb second: `[object]_[action]`
3. Past tense for completed actions
4. Never put PII in event names or properties (see Privacy)
5. Event names are copied from `app/events.json` in code, never typed by hand

---

## Standard Properties

### Recorded automatically by PostHog (do not send manually)

| PostHog field | Replaces |
|---|---|
| `uuid` | `event_id` |
| `timestamp` | `timestamp` |
| `distinct_id` / `person_id` (anonymous per-browser ID) | `user_id` |
| `$session_id` (browser events) | `session_id` |
| `$device_type`, `$browser`, `$os` (browser events) | `platform` |

### Sent by the app on every event

| Property | Type | Description | Example |
|---|---|---|---|
| `environment` | enum | Where the event came from; drives the internal-user filter | `"production"`, `"development"` |
| `source` | enum | Which side fired it | `"server"`, `"browser"` |

Internal and test traffic is excluded with PostHog's **"Filter out internal and test users"** (`environment = development`), so no separate `is_internal_test` flag is needed.

---

## Core Events (4 for MVP)

### Event 1: Document Uploaded

**Event name:** `document_uploaded`
**Fired by:** Backend (`posthog-python`), after the file is stored and its database row is committed
**Journey step:** 2 · Capture

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `document_id` | string (UUID) | Yes | Stored document ID | `"3f1c…"` |
| `file_type` | enum | Yes | File extension | `"pdf"`, `"docx"`, `"doc"`, `"xlsx"`, `"xls"`, `"csv"`, `"pptx"`, `"ppt"`, `"odt"`, `"rtf"`, `"txt"`, `"html"` |
| `file_size_kb` | integer | Yes | File size | `420` |

---

### Event 2: Document Indexed

**Event name:** `document_indexed`
**Fired by:** Backend (`posthog-python`), after indexing finishes (success **or** failure) and the status is committed
**Journey step:** 3 · Activation

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `document_id` | string (UUID) | Yes | Same ID as `document_captured` | `"3f1c…"` |
| `status` | enum | Yes | Result of indexing | `"ready"`, `"failed"` |
| `failure_reason` | enum | If failed | Why it failed (no raw error text) | `"no_text"` (scanned PDF), `"unreadable"`, `"embedding_error"` |
| `page_count` | integer | If ready | Pages, slides or sheets | `2` |
| `chunk_count` | integer | If ready | Searchable text pieces created | `14` |
| `georgian_ratio` | float 0–1 | If ready | Share of Georgian letters in the text | `0.94` |
| `legacy_text_fixed` | boolean | If ready | Pre-Unicode (keyboard-layout) Georgian was converted | `true` |
| `duration_ms` | integer | Yes | Time to index | `850` |

---

### Event 3: Question Asked

**Event name:** `question_asked`
**Fired by:** Frontend (`posthog-js`), after the server returns the answer
**Journey step:** 4 · Lookup

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `search_id` | string (UUID) | Yes | ID linking this question, answer, and source open | `"a81e…"` |
| `query_length_chars` | integer | Yes | Length of the question (never the text itself) | `27` |
| `latin_typed` | boolean | Yes | Question typed in Latin letters (translit) | `true` |

---

### Event 4: Answer Returned ⭐ 

**Event name:** `answer_returned`
**Fired by:** Backend / Frontend after `/ask` returns the answer
**Journey step:** 4 · Lookup result

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `search_id` | string (UUID) | Yes | Same ID as `question_asked` | `"a81e…"` |
| `results_count` | integer | Yes | Cited sources returned (`0` = nothing found) | `3` |
| `answer_mode` | enum | Yes | How the answer was produced | `"llm"`, `"passage_only"`, `"llm_unavailable"` |
| `latency_ms` | integer | Yes | Server time to answer | `2300` | 

---

### Event 5: Source Opened ⭐ (North Star event)

**Event name:** `source_opened` (`SOURCE_OPENED`)
**Fired by:** Frontend (`posthog-js`), after the server commits `/source-opens`
**Journey step:** 5 · North Star

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `document_id` | string (UUID) | Yes | Opened document | `"3f1c…"` |
| `search_id` | string (UUID) | Yes | The search that led here | `"a81e…"` |
| `page` | integer | Yes | Cited page, slide or sheet opened | `2` |
| `citation_rank` | integer | Yes | Position of the opened source in the answer (1 = top) | `1` |
| `retrieval_time_seconds` | integer | Yes | Seconds from submitting the question to opening the source | `9` |


**Example payload (as stored in PostHog):**

```json
{
  "event": "source_opened",
      "distinct_id": "9f82c1a4…",
      "timestamp": "2026-11-25T16:40:12Z",
      "properties": {
        "document_id": "3f1c…",
        "search_id": "a81e…",
        "page": 2,
        "citation_rank": 1,
        "retrieval_time_seconds": 9,
        "environment": "production",
        "source": "browser"
  }
}
```

---

## Event-to-Metric Mapping

### North Star Metric

**Weekly Successful Document Retrievals** = count of `source_opened` in the last 7 days, internal users excluded.

A retrieval means the user found the right document and opened it at the cited page: the moment the product saved them from searching through files by hand.

### Leading Indicators

| # | Indicator | Definition | Target |
|---|---|---|---|
| 1 | Activation | % of new users with ≥ 1 `document_indexed` (`status = ready`) and ≥ 1 `document_retrieved` within 24 h of their first `document_captured` | ≥ 40% |
| 2 | Search success | % of `document_searched` that lead to a `document_retrieved` with the same `search_id` | ≥ 50% |
| 3 | Index success | % of `document_indexed` with `status = ready` | ≥ 90% |

Supporting: median `retrieval_time_seconds` (pitch target < 15 s), share of `document_searched` with `results_count = 0`.

### AARRR Mapping

| Stage | Events | Metric |
|---|---|---|
| Acquisition | — (pageviews only) | Deferred: no signup in Sprint 1 |
| Activation | `document_captured` → `document_indexed` → `document_searched` → `document_retrieved` | Funnel conversion; Leading Indicator 1 |
| Retention | `document_retrieved` | Users with a retrieval in week N who retrieve again in week N+1 |
| Referral | — | Deferred: no workspaces or invites in Sprint 1 |
| Revenue | — | N/A for MVP |

---

## Implementation

| Event | Layer | Code location | Fires after |
|---|---|---|---|
| `document_uploaded` | Backend | `app/engine/nuskha/api.py` (upload) | File stored + DB row committed |
| `document_indexed` | Backend | `app/engine/nuskha/api.py` (upload) | Status committed (`ready` or `failed`) |
| `question_asked` | Frontend / Backend | `app/web/src/App.tsx` / `/ask` | Question submitted / accepted |
| `answer_returned` | Frontend / Backend | `app/web/src/App.tsx` (ask) | `/ask` returned 200 |
| `source_opened` | Frontend | `app/web/src/App.tsx` (open source) | `/source-opens` returned 200 |

**Rules:**
- Event names come from `app/events.json` (one file shared by engine and web). Never type them inline.
- Each event fires from exactly one layer, once.
- Browser events: inside the event handler, never in render code or effects (prevents double counting).
- Server events: after the database commit; `flush()` after capture for serverless hosting.
- Analytics never blocks or breaks the product: if PostHog is unreachable or no key is set, the event is skipped.

---

## Privacy & Compliance

**Never in event properties:**
- ❌ Filenames, document text, or extracted snippets
- ❌ Question text (only `query_length_chars` and `latin_typed`)
- ❌ Names, emails, phone numbers, personal or tax IDs (ს/კ), amounts from documents

**Use instead:** anonymous IDs (`distinct_id`, `document_id`, `search_id`), counts, booleans, enums.

**Data handling:** PostHog EU Cloud, IP anonymization on. Raw events kept 12 months. Deleting a document removes its file and database rows; analytics events keep only the anonymous ID.

---

## Reconcile Check (Lab 7)

The North Star count in PostHog must match the database:

```sql
-- Neon (database)
SELECT count(*) FROM source_opens
WHERE created_at >= now() - interval '7 days' AND owner_id NOT LIKE 'test-%';
```

```sql
-- PostHog (HogQL), internal users excluded
SELECT count() FROM events
WHERE event = 'document_retrieved' AND properties.environment = 'production'
  AND timestamp >= now() - INTERVAL 7 DAY
```

| Date | DB count | PostHog count | Difference | Note |
|---|---|---|---|---|
|23.11.2026|5|5|0| Verified live in PostHog Activity (GG)|

A small PostHog shortfall is expected: ad blockers drop some browser events. Server events (`document_captured`, `document_indexed`) are not affected.

---

## Event Volume Estimates (beta, Weeks 6–8, ~35–40 users)

| Event | Per day | Per month |
|---|---|---|
| `document_uploaded` | 15–20 | ~500 |
| `document_indexed` | 15–20 | ~500 |
| `question_asked` | 25–35 | ~900 |
| `answer_returned` | 25–35 | ~900 |
| `source_opened` | 12–18 | ~450 |
| **Total** (+ pageviews) | **~115** | **~3,500** |

Well within PostHog's free tier.

---

## Pre-Launch Checklist

- [x] Every event fires once (no duplicate frontend + backend firing)
- [x] All required properties present; names match `app/events.json`
- [x] Events appear in PostHog Activity within 60 seconds, from production
- [x] No PII in any payload (no filenames, question text, snippets)
- [ ] Internal-user filter on (`environment = development` excluded)
- [ ] Reconcile check filled in: PostHog count ≈ database count

---

## Sprint 1 Active Events & Ownership

| Step | Event | Fired by | Owner | Live |
|---|---|---|---|---|
| 2 · Capture | `document_captured` | Backend (`posthog-python`) | AJ | ☑ |
| 3 · Activation | `document_indexed` | Backend (`posthog-python`) | AJ | ☑ |
| 4 · Lookup | `document_searched` | Frontend (`posthog-js`) | AJ | ☑ |
| 5 · North Star | `document_retrieved` | Frontend (`posthog-js`) | AJ | ☑ |
| Acquisition | `user_signup_started`, `user_signup_completed` | — | — | Deferred to Sprint 2 (no accounts) |
| Retention | `user_session_started` | — | — | Dropped: PostHog retention uses existing events |
| Referral | `workspace_invite_sent` | — | — | Deferred to Sprint 2 (no workspaces) |

---

## Team Sign-off

- Giorgi Gogichashvili (`@gogicho`): Discovery & Metrics, [18.11.2026 / 23.11.2026]
- Davit Kutateladze (`@QutaInParis`): Sprint 1, [19.11.2026]
- Anri Javakhishvili (`@anrj`): [23.11.2026]

**Tech Lead approval:**
Anri Javakhishvili (`@anrj`): [X]

---

## Change Log

| Date | Version | Changes | Author |
|---|---|---|---|
| 18.11.2026 | 1.0 | Initial 8-event MVP schema (ProofVault) | Kutateladze & Co. |
| 24.11.2026 | 1.1 | Removed standard properties PostHog records automatically (`event_id`, `timestamp`, `session_id`, `platform`, `user_id`) and `workspace_id`; `is_internal_test` replaced by `environment` + PostHog internal-user filter. `document_indexed` properties changed from receipt metadata to indexing results. `user_session_started` dropped. North Star simplified to a weekly count. Added reconcile check. | Anri Javakhishvili |
| 23.11.2026 | 1.2 | Synced all 5 event names across schema, SQL/HogQL reconcile, and KPIs to match `app/events.json` of the deployed project. (`document_uploaded`, `document_indexed`, `question_asked`, `answer_returned`, `source_opened`); marked all 5 events verified live in PostHog Activity. | Giorgi Gogichashvili |

---

## Notes & Decisions

- **Why 4 events:** the Sprint 1 journey has 4 measurable steps. Signup, workspaces and invites are cut in `mvp-boundary.md`, so their events are deferred instead of defined in advance.
- **Failures are a property, not an event:** `document_indexed` fires with `status = failed` and a `failure_reason`, which powers Leading Indicator 3 without a separate `document_index_failed` event.
- **No double-click deduplication rule:** each `document_retrieved` corresponds to one committed `source_opens` row, so the database is the source of truth and the reconcile check catches inflation.