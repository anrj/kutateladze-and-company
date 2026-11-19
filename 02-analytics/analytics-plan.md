# Analytics Implementation Plan

**Team:** Kutateladze & Co.  
**Date:** 11.06.2026  
**Product:** [Working Title: ProofVault - Unified Document & Receipt Vault]  
**Timeline:** Weeks 6-8

---

## Executive Summary

**Purpose of this document:**  
This plan outlines HOW we will implement, test, monitor, and review analytics for our unified document and receipt vault MVP across Weeks 6–8.

**Key Decisions:**
- **Analytics Platform:** PostHog Cloud Free Tier (`posthog-js` + `posthog-node`) + Sentry Free Tier for error tracking
- **Implementation Approach:** Hybrid Frontend (UI & retrieval-speed timing) + Backend (verified DB mutations) tracking across 8 core events
- **Timeline:** 3 weeks (Week 6 Foundation → Week 7 Complete Events & Dashboard → Week 8 Production Launch)
- **Success Criteria:** All 8 core events live in production with `<5%` event loss, zero PII leaks, and weekly NSM reviews active by Week 8

---

## Goals & Success Criteria

### What We're Trying to Achieve

**Primary Goals:**
1. **Track North Star Metric:** Measure `Weekly Successful Document Retrievals` (`document_retrieved` in `<= 60s`) accurately
2. **Monitor AARRR Funnel:** Track Acquisition, Activation (48h `3+` indexed docs), Retention, and Referral (workspace invites)
3. **Enable Data-Driven Decisions:** Maintain a single PostHog dashboard reviewed every weekend standup
4. **Ensure Data Quality:** `<5%` event loss rate and `0` PII leaks across client and server payloads

**Success Criteria for Analytics Implementation:**

| Criterion | Target | How We'll Measure |
|-----------|--------|-------------------|
| Event Coverage | All 8 core events instrumented | Count of QA-verified events in PostHog |
| Data Accuracy | <5% event loss rate | Compare backend DB mutations (`documents` table) vs. PostHog `document_captured` count |
| Dashboard Availability | Real-time (< 1 min delay) | Measure timestamp → PostHog Live Events arrival latency |
| Team Adoption | 100% team using dashboard weekly | Weekend standup screen-share + saved `/02-analytics/weekly-reviews/` notes |
| Performance Impact | < 50ms added latency | Browser Network timing & async non-blocking SDK calls |

---

## Analytics Architecture

### High-Level Design

**Data Flow:**

    [User Action in React App]
              |
              +---> UI / Speed Events (posthog-js) ----> [Client Validation & Queue]
              |                                                    |
              v                                                    v
    [Express API + App DB] ---> Mutation Events (posthog-node) ---> [PostHog Cloud Ingestion]
                                                                           |
                                       +-----------------------------------+
                                       |                                   |
                                       v                                   v
                            [PostHog Event Store]               [Alerts -> Team Group Chat]
                            (12-mo retention, no PII)
                                       |
                                       v
                            [PostHog MVP Dashboard]
                            (NSM, AARRR Funnel, Cohorts)

**Description:**  
When a user opens signup, searches, or retrieves a document in the React frontend, `posthog-js` validates the payload, strips PII, and sends the event asynchronously to PostHog Cloud. When a user completes signup, uploads a file, saves indexed metadata, or sends a workspace invite, our Express API waits for DB persistence and fires the event via `posthog-node` so ad blockers cannot drop core business events.

---

### Technology Stack

| Component | Tool/Service | Version | Justification |
|-----------|-------------|---------|---------------|
| **Analytics Platform** | PostHog Cloud | Free Tier (1M events/mo) | Combines event ingestion, funnels, retention cohorts, and live debugging in one free tool |
| **Data Storage** | PostHog Event Store + App DB | Managed Cloud | 12-month raw event retention in PostHog without maintaining a separate warehouse |
| **Visualization** | PostHog Dashboards | Built-in | Native 7-day rolling NSM trend, AARRR funnels, and cohort tables |
| **Client SDK** | `posthog-js` | `^1.x` | Lightweight async queue/batching for React (`vite` SPA) |
| **Backend Tracking** | `posthog-node` | `^4.x` | Reliable server-side tracking from Express API (`server.ts`) |
| **Error Monitoring** | Sentry | Free Tier (5K errors/mo) | Captures upload, OCR, and API failures separately from product analytics |

---

### Alternative Solutions Considered

| Solution | Pros | Cons | Why Not Chosen |
|----------|------|------|----------------|
| **Segment + Mixpanel + BigQuery** | Industry standard separation of collection & BI | Multiple integrations to wire; Mixpanel paid tiers scale fast | Over-engineered for a 3-week MVP build |
| **Google Analytics 4 (GA4)** | Free, widely known | Weak backend mutation tracking, ~30% ad-blocker loss, cumbersome custom event funnel queries | Poor fit for `document_indexed` and `< 15s` retrieval diagnostics |
| **Custom DB Event Table Only** | Full control, zero third-party SDK | Requires building funnels, cohort SQL, and charts from scratch | Wastes Week 6–7 engineering hours needed for core product |

---

## Implementation Timeline

### Week-by-Week Plan

**Week 6: Foundation**
- [ ] Set up PostHog Cloud organization (`dev` + `prod` projects) and Sentry project
- [ ] Configure `.env.development`, `.env.staging`, and `.env.production` keys
- [ ] Install `posthog-js` in React frontend and `posthog-node` in Express backend
- [ ] Build `trackEvent()` wrapper with schema validation & PII guards
- [ ] Implement first core events (`user_signup_started`, `user_signup_completed`, `document_captured`, `document_retrieved`)
- [ ] Verify events appearing in PostHog Live Events (dev/staging)

**Week 7: Core Events & Dashboard**
- [ ] Implement remaining 4 events (`document_indexed`, `document_searched`, `workspace_invite_sent`, `user_session_started`)
- [ ] Add automated unit tests for required properties and PII exclusion
- [ ] Build PostHog NSM Trend + AARRR Funnel + Leading Indicators dashboard
- [ ] Test all 8 events end-to-end in staging across desktop & mobile web

**Week 8: Polish & Launch**
- [ ] Deploy production build with production PostHog keys
- [ ] Run flagged production smoke test (`is_internal_test: true`) and monitor 72 hours
- [ ] Configure pipeline & anomaly alerts to team group chat
- [ ] Run 20-minute team walkthrough on reading dashboard widgets
- [ ] Complete first Weekend Standup analytics review and log `/02-analytics/weekly-reviews/week-08.md`

---

### Daily Breakdown (Week 6 Foundation)

| Day | Tasks | Owner | Estimated Hours |
|-----|-------|-------|-----------------|
| Monday | Provision PostHog (`dev`/`prod`) & Sentry projects; add `.env.example` vars | Anri (`@anrj`) | 2 |
| Tuesday | Install `posthog-js` & `posthog-node`; write shared `trackEvent` wrappers | Anri (`@anrj`), Nikoloz (`@Tuxa4Life`) | 3 |
| Wednesday | Instrument `user_signup_started` (FE) & `user_signup_completed` (BE) | Nikoloz (`@Tuxa4Life`), Anri (`@anrj`) | 3 |
| Thursday | Instrument `document_captured` (BE) & `document_retrieved` (FE) | Anri (`@anrj`), Nikoloz (`@Tuxa4Life`) | 3 |
| Friday | Verify payloads in PostHog Live Events; tag team QA IDs `is_internal_test: true` | Davit (`@QutaInParis`), Giorgi (`@gogicho`) | 2 |

---

## Technical Implementation

### Frontend (Web / Mobile Web)

**Framework:** React + TypeScript (Vite)

**Installation Steps:**

    npm install posthog-js

**Configuration (`src/lib/analytics.ts`):**

    import posthog from 'posthog-js';

    const POSTHOG_KEY = import.meta.env.VITE_POSTHOG_KEY;
    const POSTHOG_HOST = import.meta.env.VITE_POSTHOG_HOST || 'https://us.i.posthog.com';

    export function initAnalytics(): void {
      if (!POSTHOG_KEY) return;
      posthog.init(POSTHOG_KEY, {
        api_host: POSTHOG_HOST,
        autocapture: false,
        capture_pageview: false,
        mask_all_text: true,
        mask_all_element_attributes: true,
      });
    }

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
        // Never block UI interactions if analytics fails
      }
    }

**Usage Example (Retrieval Timing):**

    import { trackEvent } from '../lib/analytics';

    function onOpenDocument(
      docId: string,
      category: string,
      searchStartedAtMs: number,
      searchId?: string
    ) {
      const retrievalSeconds = Math.max(
        0,
        Math.round((Date.now() - searchStartedAtMs) / 1000)
      );

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

### Backend (API / Server)

**Framework:** Node.js + Express (`server.ts`)

**Installation Steps:**

    npm install posthog-node

**Configuration (`server/analytics.ts`):**

    import { PostHog } from 'posthog-node';

    const client = process.env.POSTHOG_SERVER_KEY
      ? new PostHog(process.env.POSTHOG_SERVER_KEY, {
          host: process.env.POSTHOG_HOST || 'https://us.i.posthog.com',
          flushAt: 20,
          flushInterval: 10000,
        })
      : null;

    export function trackServerEvent(
      userId: string,
      eventName: string,
      properties: Record<string, string | number | boolean>
    ): void {
      if (!client) return;
      client.capture({
        distinctId: userId,
        event: eventName,
        properties: {
          event_id: crypto.randomUUID(),
          timestamp: new Date().toISOString(),
          platform: 'web',
          app_version: process.env.APP_VERSION ?? '0.1.0',
          environment: process.env.NODE_ENV ?? 'development',
          ...properties,
        },
      });
    }

    export async function shutdownAnalytics(): Promise<void> {
      await client?.shutdown();
    }

**Usage Example (`server.ts`):**

    app.post('/api/documents/:id/index', async (req, res) => {
      const doc = await saveDocumentMetadata(req.params.id, req.body);

      trackServerEvent(req.user.id, 'document_indexed', {
        workspace_id: doc.workspaceId,
        document_id: doc.id,
        document_category: doc.category,
        indexing_method: doc.indexingMethod,
        has_vendor: Boolean(doc.vendor),
        has_document_date: Boolean(doc.documentDate),
        has_amount: doc.amount !== null,
        has_expiry_date: Boolean(doc.expiryDate),
        metadata_complete: Boolean(doc.vendor && doc.documentDate),
        is_internal_test: req.user.isInternalTest ?? false,
      });

      res.json(doc);
    });

---

### Environment Configuration

**Development (`.env.development`):**

    VITE_POSTHOG_KEY=phc_dev_key_placeholder
    POSTHOG_SERVER_KEY=phc_dev_key_placeholder
    POSTHOG_HOST=https://us.i.posthog.com
    VITE_APP_VERSION=0.1.0-dev

**Staging (`.env.staging`):**

    VITE_POSTHOG_KEY=phc_staging_key_placeholder
    POSTHOG_SERVER_KEY=phc_staging_key_placeholder
    POSTHOG_HOST=https://us.i.posthog.com
    VITE_APP_VERSION=0.1.0-rc

**Production (`.env.production`):**

    VITE_POSTHOG_KEY=phc_prod_key_placeholder
    POSTHOG_SERVER_KEY=phc_prod_key_placeholder
    POSTHOG_HOST=https://us.i.posthog.com
    VITE_APP_VERSION=0.1.0

---

## Event Implementation Checklist

### Event 1: `user_signup_started`
- [x] **Defined:** Documented in `/02-analytics/event-schema.md`
- [ ] **Instrumented:** Code written to fire event
- [ ] **Tested (Dev):** Fires on signup screen / invite open
- [ ] **Tested (Staging):** Verified in PostHog staging
- [ ] **Validated & No PII:** `signup_method`, `entry_source` only
- [ ] **Deployed:** Live in production

**Location:** `src/pages/Signup.tsx`  
**Owner:** Nikoloz (`@Tuxa4Life`)  
**Status:** Planned (Week 6)

---

### Event 2: `user_signup_completed`
- [x] **Defined:** Documented in `/02-analytics/event-schema.md`
- [ ] **Instrumented:** Fired on backend after user + workspace creation
- [ ] **Tested (Dev):** Verified in dev
- [ ] **Tested (Staging):** Verified in staging
- [ ] **Validated & No PII:** `user_segment`, `joined_existing_workspace`, `time_to_complete_seconds`
- [ ] **Deployed:** Live in production

**Location:** `server.ts` (`POST /api/auth/signup`)  
**Owner:** Anri (`@anrj`)  
**Status:** Planned (Week 6)

---

### Event 3: `document_captured`
- [x] **Defined:** Documented in `/02-analytics/event-schema.md`
- [ ] **Instrumented:** Fired on backend after file upload completes
- [ ] **Tested (Dev):** Verified in dev
- [ ] **Tested (Staging):** Verified in staging
- [ ] **Validated & No PII:** `capture_channel`, `file_type`, `contributor_role` (no filenames)
- [ ] **Deployed:** Live in production

**Location:** `server.ts` (`POST /api/documents/upload`)  
**Owner:** Anri (`@anrj`)  
**Status:** Planned (Week 6)

---

### Event 4: `document_indexed`
- [x] **Defined:** Documented in `/02-analytics/event-schema.md`
- [ ] **Instrumented:** Fired on backend when OCR/manual metadata saves
- [ ] **Tested (Dev):** Verified in dev
- [ ] **Tested (Staging):** Verified in staging
- [ ] **Validated & No PII:** Boolean flags (`has_vendor`, `metadata_complete`) only
- [ ] **Deployed:** Live in production

**Location:** `server.ts` (`POST /api/documents/:id/index`)  
**Owner:** Anri (`@anrj`)  
**Status:** Planned (Week 7)

---

### Event 5: `document_searched`
- [x] **Defined:** Documented in `/02-analytics/event-schema.md`
- [ ] **Instrumented:** Fired on frontend when search/filter executes
- [ ] **Tested (Dev):** Verified in dev
- [ ] **Tested (Staging):** Verified in staging
- [ ] **Validated & No PII:** `query_length_chars` & `results_count` (no raw search text)
- [ ] **Deployed:** Live in production

**Location:** `src/components/VaultSearch.tsx`  
**Owner:** Nikoloz (`@Tuxa4Life`)  
**Status:** Planned (Week 7)

---

### Event 6: `document_retrieved` (NSM Event)
- [x] **Defined:** Documented in `/02-analytics/event-schema.md`
- [ ] **Instrumented:** Fired on frontend when user opens/exports/shares matching doc
- [ ] **Tested (Dev):** Verified `retrieval_time_seconds` calculation
- [ ] **Tested (Staging):** Verified NSM dashboard increments
- [ ] **Validated & No PII:** `result_action`, `retrieval_time_seconds`, `document_category`
- [ ] **Deployed:** Live in production

**Location:** `src/components/DocumentViewer.tsx`  
**Owner:** Nikoloz (`@Tuxa4Life`)  
**Status:** Planned (Week 6)

---

### Event 7: `workspace_invite_sent`
- [x] **Defined:** Documented in `/02-analytics/event-schema.md`
- [ ] **Instrumented:** Fired on backend when invite link/token is generated
- [ ] **Tested (Dev):** Verified in dev
- [ ] **Tested (Staging):** Verified in staging
- [ ] **Validated & No PII:** `invite_role`, `invite_channel`, `current_member_count`
- [ ] **Deployed:** Live in production

**Location:** `server.ts` (`POST /api/workspaces/invites`)  
**Owner:** Anri (`@anrj`)  
**Status:** Planned (Week 7)

---

### Event 8: `user_session_started`
- [x] **Defined:** Documented in `/02-analytics/event-schema.md`
- [ ] **Instrumented:** Fired on frontend once per 30-minute authenticated session
- [ ] **Tested (Dev):** Verified debouncing works
- [ ] **Tested (Staging):** Verified cohort retention populates
- [ ] **Validated & No PII:** `days_since_signup`, `days_since_last_visit`, `workspace_doc_count`
- [ ] **Deployed:** Live in production

**Location:** `src/App.tsx`  
**Owner:** Nikoloz (`@Tuxa4Life`)  
**Status:** Planned (Week 7)

---

## Dashboard Design

### North Star Metric Dashboard

**Primary View Metrics to Display:**
1. **Current NSM Value:** Weekly Successful Document Retrievals (`document_retrieved` in `<= 60s`, excluding `is_internal_test`)
2. **Week-over-Week Change:** `%` delta vs. prior 7 days
3. **Trend Line:** 30-day rolling 7-day trend vs. MVP Target (`75/wk`)
4. **Breakdown by Segment:** `household` vs. `small_office_admin` / `accountant` / `business_owner`

**Refresh Rate:** Real-time Live Events + 15-minute insight refresh  
**Access:** All 4 Kutateladze & Co. members

**Mockup:**

    +-------------------------------------------------------------------+
    |  North Star Metric: Weekly Successful Document Retrievals         |
    |                                                                   |
    |  Current (7d): 78       Target: 75/wk       WoW: +18% vs LW       |
    |                                                                   |
    |  [7-day rolling line graph with 35-min & 75-target markers]       |
    |                                                                   |
    |  Breakdown by Segment:                                            |
    |  - Small Office / Accountants: 58 retrievals (median 11s)         |
    |  - Households:                 20 retrievals (median 14s)         |
    +-------------------------------------------------------------------+

---

### AARRR Funnel Dashboard

**Stages & Targets:**
1. **Acquisition:** `user_signup_started` → `user_signup_completed` (Target: `>= 65%`)
2. **Activation:** `user_signup_completed` → `3+ document_indexed` within 48h (Target: `>= 60%`)
3. **Retention:** Activated users firing `user_session_started` or `document_retrieved` in Week N+1 (Target: `>= 35%`)
4. **Referral:** Active workspaces firing `workspace_invite_sent` (Target: `>= 35%`)
5. **Revenue:** N/A for free MVP beta (V2 B2B workspace tier)

**Alerts:**  
If 48h Activation drops below `40%` or `metadata_complete` falls below `75%`, trigger a team chat alert.

---

### Weekly Review Dashboard

**Purpose:** Reviewed together at every Weekend Standup

**Sections:**
1. **North Star Metric:** 7-day retrievals, target gap, WoW trend
2. **Leading Indicators:** 48h Activation Rate, `metadata_complete` %, Multi-Contributor Workspace %
3. **Retrieval Speed & Zero-Result Searches:** Median `retrieval_time_seconds` (`< 15s` goal) & `%` of `document_searched` with `results_count = 0`
4. **AARRR Funnel & Cohorts:** Drop-off points and Week-2 retention by user segment

---

## Data Quality & Monitoring

### Validation Rules

**Pre-send validation (in code):**
- Enforce all 9 standard properties and required custom fields
- Validate enum values against allowed sets (`document_category`, `result_action`, `user_segment`)
- Enforce `is_internal_test` flag on team accounts
- Reject/strip forbidden keys (`email`, `phone`, `file_name`, `search_query`, `raw_text`)

**Example Validation Function:**

    const FORBIDDEN_PII_KEYS = new Set([
      'email',
      'phone',
      'full_name',
      'file_name',
      'filename',
      'search_query',
      'raw_text',
    ]);

    export function assertSafePayload(properties: Record<string, unknown>): void {
      for (const key of Object.keys(properties)) {
        if (FORBIDDEN_PII_KEYS.has(key.toLowerCase())) {
          throw new Error(`Blocked PII analytics property: ${key}`);
        }
      }
    }

---

### Error Handling

- **Client-side errors:** Log warnings in `development`, capture unexpected exceptions in Sentry in `production`, rely on `posthog-js` localStorage retry queue, and never block upload or search UI.
- **Server-side errors:** Log structured error in Express logs, alert Tech Lead if delivery failure rate exceeds `10%`, and rely on `posthog-node` flush/retry queue.

---

### Monitoring & Alerts

**Daily Checks (Weeks 6–8):**
- [ ] Event volume matches active beta usage (no silent drop to `0`)
- [ ] Zero Sentry errors on `/api/documents/*` upload/index routes
- [ ] Dashboard loads cleanly with `is_internal_test = false` filter active

**Alerts to Set Up:**

| Condition | Alert Channel | Recipient |
|-----------|--------------|-----------|
| Zero events received for >6 hours | Team Group Chat | Anri (`@anrj`) |
| `metadata_complete = false` exceeds `25%` | Team Group Chat | Anri (`@anrj`) & Nikoloz (`@Tuxa4Life`) |
| NSM drops `> 40%` week-over-week | Team Group Chat | Giorgi (`@gogicho`) |
| Any PII key error caught in Sentry | Immediate Direct Ping | Entire team |

---

## Testing Strategy

### Unit Tests

Test `trackEvent()`, `assertSafePayload()`, and `retrieval_time_seconds` calculation:

    import { describe, expect, it } from 'vitest';
    import { assertSafePayload } from '../src/lib/analytics';

    describe('analytics privacy guard', () => {
      it('allows safe document_retrieved properties', () => {
        expect(() =>
          assertSafePayload({
            document_id: 'doc_4481a9c0',
            document_category: 'warranty',
            result_action: 'opened',
            retrieval_time_seconds: 9,
          })
        ).not.toThrow();
      });

      it('rejects PII keys such as email or search_query', () => {
        expect(() =>
          assertSafePayload({ search_query: 'boiler receipt' })
        ).toThrow('Blocked PII analytics property');
      });
    });

---

### Integration & End-to-End Testing

**Scenario:** Beta tester signs up, indexes 3 documents, searches, and retrieves a warranty.

**Steps:**
1. Open signup page → verify `user_signup_started` fires once.
2. Complete registration → verify `user_signup_completed` fires from Express API.
3. Upload & confirm metadata on 3 documents → verify 3× `document_captured` and 3× `document_indexed`.
4. Search by category/keyword → verify `document_searched` logs `query_length_chars` (not text) and `results_count`.
5. Open matching warranty → verify `document_retrieved` logs `retrieval_time_seconds` and increments NSM by `1`.

---

## Team Responsibilities

### Analytics Owner: Giorgi Gogichashvili (`@gogicho`)
- Owns NSM & Leading Indicator definitions
- Builds PostHog dashboard widgets and leads Weekend Standup metrics review
- Connects metric drops directly to user check-in questions (`3–4 hrs/week`)

### Tech Lead & Backend Instrumentation: Anri Javakhishvili (`@anrj`)
- Configures PostHog (`dev`/`prod`) & Sentry projects and environment secrets
- Implements `posthog-node` server events and PII validation checks
- Reviews all instrumentation PRs (`4–5 hrs/week` during Weeks 6–7)

### Frontend Instrumentation: Nikoloz Tukhashvili (`@Tuxa4Life`)
- Implements `posthog-js`, `trackEvent()` wrapper, and client retrieval-speed timer
- Instruments `user_signup_started`, `document_searched`, `document_retrieved`, and `user_session_started` (`3–4 hrs/week` during Weeks 6–7)

### QA & Delivery Lead: Davit Kutateladze (`@QutaInParis`)
- Executes dev/staging/prod QA checklists across desktop & mobile web
- Maintains `is_internal_test` filter hygiene and archives weekly review notes (`2–3 hrs/week`)

---

## Privacy & Compliance

### Data Collection Notice

**Where:** Signup screen & Settings page

**Language:**  
"We collect anonymized product usage events (such as upload counts, search speed, and feature clicks) to improve document retrieval speed. We never track your document contents, filenames, search text, or personal contact details in analytics."

### User Consent & Data Security

- **Consent:** Beta consent notice at signup + Settings toggle to pause client-side analytics; PostHog Session Replay masks all form inputs and document preview containers.
- **In Transit & At Rest:** All events sent over HTTPS; API keys stored strictly in environment variables/host secrets; `dev` and `prod` projects separated; access restricted to the 4 team members.

---

## Budget & Costs

### Free Tier Limits

| Service | Free Tier Limit | Expected MVP Usage | Headroom |
|---------|----------------|--------------------|----------|
| PostHog Cloud | 1,000,000 events/month | ~2,220 events/month | `99.7%` headroom |
| PostHog Session Replay | 5,000 recordings/month | ~150 recordings/month | `97%` headroom |
| Sentry Error Tracking | 5,000 errors/month | `< 200` errors/month | `96%` headroom |

**Total Cost:** `$0/month`

### Upgrade Triggers

We would only consider upgrading if beta scale exceeds `500,000+` events/month or `5,000` monthly session replays (unlikely during the semester). Any paid upgrade requires unanimous team approval.

---

## Launch Checklist

**Pre-Launch (Week 7–8):**
- [ ] All 8 core events implemented and QA-verified in staging
- [ ] PostHog MVP dashboard built with all 6 widgets and `is_internal_test = false` filter
- [ ] Alerts active in team group chat
- [ ] Zero PII leaks verified in staging payloads
- [ ] Privacy notice visible on signup screen

**Launch Day (Week 8):**
- [ ] Deploy with production PostHog keys
- [ ] Run 1 flagged smoke test and verify arrival in Prod Live Events within 60s
- [ ] Verify team test flag keeps test events out of NSM count

**Post-Launch (First Week):**
- [ ] Check event counts vs. DB document records daily
- [ ] Hold first Weekend Standup dashboard review
- [ ] Log findings & Week 9 experiment in `/02-analytics/weekly-reviews/week-08.md`

---

## Maintenance Plan

- **Every Weekend Standup (30 min):** Review 7-day NSM (`75/wk` target), 48h Activation (`60%`), Metadata Completeness (`90%`), Multi-Contributor rate (`35%`), median retrieval speed (`< 15s`), and zero-result searches; assign 1–2 action items.
- **Monthly (1 hr):** Audit schema compatibility, compare PostHog event counts vs. DB rows (`< 5%` discrepancy), spot-check payloads for PII, and archive KPI summary.
- **As Needed:** Update `event-schema.md` before shipping any new Week 9+ feature.

---

## Success Metrics (for Analytics Itself)

| Metric | Target | Current | Notes |
|--------|--------|---------|-------|
| Event Coverage | 100% of 8 core events | 0% (planned Wk 6–7) | Tracked via checklist above |
| Data Accuracy | < 5% loss vs. DB rows | TBD (Wk 7 QA) | Server-side tracking minimizes ad-blocker loss |
| Dashboard Usage | Reviewed every weekend + ad-hoc checks | TBD (Wk 8) | Required standup agenda item |
| Time to Insight | < 5 min to diagnose funnel drop-off | TBD (Wk 8) | Pre-built AARRR + zero-result search widgets |

---

## Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **Ad blockers block `posthog-js`** | High (~30%) | Medium | Fire core mutation events (`signup_completed`, `document_captured`, `document_indexed`, `workspace_invite_sent`) from Express backend (`posthog-node`) |
| **Developers skip tracking on new features** | Medium | High | Block PR merges without updated `event-schema.md` check & staging verification |
| **Low household search frequency in 2-week beta** | Medium | Medium | Segment Household vs. Small Office on dashboard and recruit weekly-volume admins (`04, 11, 14, 15`) |
| **Team ignores dashboard during crunch** | Low | High | Open every Weekend Standup with a 5-min PostHog dashboard check |

---

## Future Improvements (Post-MVP)

**Week 9+:**
1. **Onboarding A/B Experiment:** Test `"15-second retrieval challenge"` prompt after 3rd indexed document
2. **Pattern #5 Expiry Instrumentation:** Add `expiry_alert_sent` and `renewal_marked_complete` events
3. **Weekly Group-Chat Digest:** Automated PostHog webhook summary before weekend standup
4. **Demo Day Proof Export:** Export cohort retention and retrieval-speed charts (`20 min–2 hrs` → `< 15s`)

---

## References

- **Event Schema:** `/02-analytics/event-schema.md`
- **North Star Metric:** `/02-analytics/north-star-metric.md`
- **Problem Statement:** `/01-discovery/synthesis/final-problem-statement.md`
- **Team Contract:** `/00-foundation/team-contract.md`

---

## Team Sign-off

- Giorgi Gogichashvili (`@gogicho`) - Analytics Lead - 18.11.2026
- Davit Kutateladze (`@QutaInParis`) - [19.11.2026]
-
-

**Tech Lead approval:**  
Anri Javakhishvili (`@anrj`) - 

---

## Change Log

| Date | Version | Changes | Author |
|------|---------|---------|--------|
| [FILL IN: Date] | 1.0 | Initial MVP analytics plan for ProofVault | Kutateladze & Co. |