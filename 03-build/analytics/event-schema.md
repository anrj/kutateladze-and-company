# Event Schema Design

**Team:** Kutateladze & Co.
**Date:** 26.11.2026
**Product:** Nuskha (ნუსხა): Georgian document search & Q&A
**Version:** 1.3 (MVP, Sprint 2)

---

## Overview

This document defines the analytics events for the MVP journey: **upload → index → ask → answer → open the cited source**. Each event maps to one step of `03-build/mvp-boundary.md`. Properties contain no PII.

Version 1.3 matches the event names and properties the code actually sends (`app/events.json`, `app/engine/nuskha/api.py`, `app/web/src/App.tsx`). If the code and this file ever disagree, fix one of them in the same commit.

**Purpose:**
- Power the North Star Metric (`Weekly Successful Document Retrievals`) and 3 Leading Indicators
- Show where users drop off: upload fails to index, question finds nothing, answer found but no source opened
- Keep instrumentation lean: **5 events**, no properties that PostHog already records

---

## Event Naming Conventions

**Format:** `object_action` (snake_case, past tense)

- ✅ `document_uploaded`, `source_opened`
- ❌ `DocumentUploaded` (wrong case), `upload_document` (wrong order), `source_open` (wrong tense)

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
| `environment` | enum | Where the event came from; drives the internal-user filter. Server: `APP_ENV`; browser: the Vite build mode | `"production"`, `"development"` |
| `source` | enum | Which side fired it | `"server"`, `"browser"` |

Internal and test traffic is excluded with PostHog's **"Filter out internal and test users"** (`environment = development`), so no separate `is_internal_test` flag is needed.

---

## Core Events (5 for MVP)

### Event 1: Document Uploaded

**Event name:** `document_uploaded`
**Fired by:** Backend (`posthog-python`), after the file is stored and its database row is committed
**Journey step:** 2 · Capture

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `document_id` | string (UUID) | Yes | Stored document ID | `"3f1c…"` |
| `file_type` | enum | Yes | File extension | `"pdf"`, `"docx"`, `"doc"`, `"xlsx"`, `"xls"`, `"csv"`, `"pptx"`, `"ppt"`, `"odt"`, `"rtf"`, `"txt"`, `"html"` |
| `size_bytes` | integer | Yes | File size | `430080` |

---

### Event 2: Document Indexed

**Event name:** `document_indexed`
**Fired by:** Backend, after indexing **succeeds** and the `ready` status is committed. A failed index fires no event; the `documents` row keeps `status = failed` and the error.
**Journey step:** 3 · Activation

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `document_id` | string (UUID) | Yes | Same ID as `document_uploaded` | `"3f1c…"` |
| `page_count` | integer | Yes | Pages, slides or sheets | `2` |
| `chunk_count` | integer | Yes | Searchable text pieces created | `14` |
| `georgian_ratio` | float 0–1 | Yes | Share of Georgian letters in the text | `0.94` |
| `legacy_font_fixed` | boolean | Yes | Pre-Unicode (keyboard-layout) Georgian was converted | `true` |
| `duration_ms` | integer | Yes | Time to index | `850` |

---

### Event 3: Answer Returned

**Event name:** `answer_returned`
**Fired by:** Backend, after the question and its answer are committed to the `questions` table
**Journey step:** 4 · Lookup (server side, not affected by ad blockers)

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `question_id` | string (UUID) | Yes | ID linking this question to a source open | `"a81e…"` |
| `citation_count` | integer | Yes | Cited sources returned (`0` = nothing found) | `3` |
| `latency_ms` | integer | Yes | Server time to answer | `2300` |
| `mode` | enum | Yes | How the answer was produced: `llm` (AI answer), `none` (no AI configured, or nothing found), `unavailable` (AI busy, best passage shown) | `"llm"` |

---

### Event 4: Question Asked

**Event name:** `question_asked`
**Fired by:** Frontend (`posthog-js`), after `/ask` returns 200
**Journey step:** 4 · Lookup (browser side)

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `question_id` | string (UUID) | Yes | Same ID as `answer_returned` | `"a81e…"` |
| `question_length` | integer | Yes | Length of the question in characters (never the text itself) | `27` |
| `latin_typed` | boolean | Yes | Question typed in Latin letters (translit) | `true` |

---

### Event 5: Source Opened ⭐ (North Star event)

**Event name:** `source_opened`
**Fired by:** Frontend, after `/source-opens` returns 200 (the `source_opens` row is committed)
**Journey step:** 5 · North Star

| Property | Type | Required | Description | Example |
|---|---|---|---|---|
| `source_open_id` | integer | Yes | ID of the committed `source_opens` row | `57` |
| `question_id` | string (UUID) | Yes | The question that led here | `"a81e…"` |
| `document_id` | string (UUID) | Yes | Opened document | `"3f1c…"` |
| `page` | integer | Yes | Cited page, slide or sheet opened | `2` |
| `citation_rank` | integer | Yes | Number of the opened source in the answer (1 = first) | `1` |

**Example payload (as stored in PostHog):**

```json
{
  "event": "source_opened",
  "distinct_id": "9f82c1a4…",
  "timestamp": "2026-11-26T12:40:12Z",
  "properties": {
    "source_open_id": 57,
    "question_id": "a81e…",
    "document_id": "3f1c…",
    "page": 2,
    "citation_rank": 1,
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
| 1 | Activation | % of new users with ≥ 1 `document_indexed` and ≥ 1 `source_opened` within 24 h of their first `document_uploaded` | ≥ 40% |
| 2 | Search success | % of `answer_returned` that lead to a `source_opened` with the same `question_id` | ≥ 50% |
| 3 | Index success | `document_indexed` count ÷ `document_uploaded` count (failures fire no event) | ≥ 90% |

Supporting: share of `answer_returned` with `citation_count = 0`; share with `mode = unavailable`; median `latency_ms`.

### AARRR Mapping

| Stage | Events | Metric |
|---|---|---|
| Acquisition | — (pageviews only) | Deferred: no signup in the MVP |
| Activation | `document_uploaded` → `document_indexed` → `question_asked` → `source_opened` | Funnel conversion; Leading Indicator 1 |
| Retention | `source_opened` | Users with a retrieval in week N who retrieve again in week N+1 |
| Referral | — | Deferred: no workspaces or invites in the MVP |
| Revenue | — | N/A for MVP |

---

## Implementation

| Event | Layer | Code location | Fires after |
|---|---|---|---|
| `document_uploaded` | Backend | `app/engine/nuskha/api.py` (`POST /documents`) | File stored + DB row committed |
| `document_indexed` | Backend | `app/engine/nuskha/api.py` (`POST /documents`) | `ready` status committed |
| `answer_returned` | Backend | `app/engine/nuskha/api.py` (`POST /ask`) | `questions` row committed |
| `question_asked` | Frontend | `app/web/src/App.tsx` (`ask`) | `/ask` returned 200 |
| `source_opened` | Frontend | `app/web/src/App.tsx` (`openSource`) | `/source-opens` returned 200 |

**Rules:**
- Event names come from `app/events.json` (one file shared by engine and web). Never type them inline.
- Each event fires from exactly one layer, once.
- Browser events: inside the event handler, never in render code or effects (prevents double counting).
- Server events: after the database commit; `flush()` after capture for serverless hosting.
- Analytics never blocks or breaks the product: if PostHog is unreachable or no key is set, the event is skipped.
- The browser key (`VITE_POSTHOG_KEY`) is baked in at build time: after setting it on the web host, redeploy.

---

## Privacy & Compliance

**Never in event properties:**
- ❌ Filenames, document text, or extracted snippets
- ❌ Question text (only `question_length` and `latin_typed`)
- ❌ Names, emails, phone numbers, personal or tax IDs (ს/კ), amounts from documents

**Use instead:** anonymous IDs (`distinct_id`, `document_id`, `question_id`, `source_open_id`), counts, booleans, enums.

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
WHERE event = 'source_opened' AND properties.environment = 'production'
  AND timestamp >= now() - INTERVAL 7 DAY
```

| Date | DB count | PostHog count | Difference | Note |
|---|---|---|---|---|
|23.11.2026|5|5|0| Verified live in PostHog Activity (GG)|

A small PostHog shortfall is expected: ad blockers drop some browser events. Server events (`document_uploaded`, `document_indexed`, `answer_returned`) are not affected, so `answer_returned` against the `questions` table is a second, ad-blocker-proof check.

---

## Event Volume Estimates (beta, Weeks 6–8, ~35–40 users)

| Event | Per day | Per month |
|---|---|---|
| `document_uploaded` | 15–20 | ~500 |
| `document_indexed` | 15–20 | ~500 |
| `answer_returned` | 25–35 | ~900 |
| `question_asked` | 25–35 | ~900 |
| `source_opened` | 12–18 | ~450 |
| **Total** (+ pageviews) | **~130** | **~3,900** |

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

## Active Events & Ownership

| Step | Event | Fired by | Owner | Live |
|---|---|---|---|---|
| 2 · Capture | `document_uploaded` | Backend (`posthog-python`) | AJ | ☑ |
| 3 · Activation | `document_indexed` | Backend (`posthog-python`) | AJ | ☑ |
| 4 · Lookup | `answer_returned` | Backend (`posthog-python`) | AJ | ☑ |
| 4 · Lookup | `question_asked` | Frontend (`posthog-js`) | AJ | ☑ |
| 5 · North Star | `source_opened` | Frontend (`posthog-js`) | AJ | ☑ |
| Acquisition | `user_signup_started`, `user_signup_completed` | — | — | Deferred (no accounts) |
| Retention | `user_session_started` | — | — | Dropped: PostHog retention uses existing events |
| Referral | `workspace_invite_sent` | — | — | Deferred (no workspaces) |

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
| 26.11.2026 | 1.3 | Properties and the remaining sections matched to the events the code sends: `document_captured` → `document_uploaded` (`file_size_kb` → `size_bytes`); `document_searched` split into `answer_returned` (server) and `question_asked` (browser); `document_retrieved` → `source_opened` (North Star). `search_id` → `question_id`; `retrieval_time_seconds` dropped; `legacy_text_fixed` → `legacy_font_fixed`. `document_indexed` fires only on success, so `status`/`failure_reason` removed and Index success is uploaded vs indexed. | Anri Javakhishvili |

---

## Notes & Decisions

- **Why 5 events:** the journey has 4 measurable steps, and the question step is captured on both sides: `answer_returned` from the server (reliable, ad-blocker-proof) and `question_asked` from the browser (carries `latin_typed`).
- **Failures are counted, not evented:** a failed index fires nothing; Index success is `document_indexed` ÷ `document_uploaded`, and the reason lives in the `documents.error` column.
- **No double-click deduplication rule:** each `source_opened` corresponds to one committed `source_opens` row, so the database is the source of truth and the reconcile check catches inflation.
