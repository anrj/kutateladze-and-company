# North Star Metric Definition

**Team:** Kutateladze & Co.  
**Product:** [Working Title: ProofVault - Unified Document & Receipt Vault]  
**Problem Statement Reference:** See `/01-discovery/synthesis/final-problem-statement.md`

---

## What is a North Star Metric?

Your North Star Metric (NSM) is the **single metric** that best captures the core value your product delivers to users. It's the one number that, if it goes up, indicates your product is succeeding at solving the validated problem you identified in your synthesis.

---

## Our North Star Metric

### The Metric

**North Star Metric:** Weekly Successful Document Retrievals

**Formula/Calculation:**  
`COUNT(event_id WHERE event_name = 'document_retrieved' AND retrieval_time_seconds <= 60 AND result_action IN ('opened', 'exported', 'shared') AND is_internal_test = false AND timestamp WITHIN last_7_days)`

---

### Why This Metric?

**Connection to Problem Statement:**  
Our validated problem statement shows that household document keepers and small-organization admins already store documents in drawers, Google Drive, and WhatsApp-yet lose 2–8 hours/month and 140–500+ GEL per incident because they cannot find when emergency hits. Tracking **Weekly Successful Document Retrievals** (finding and opening/exporting an indexed document in under 60 seconds) directly measures whether we replaced chaotic searching with instant, reliable proof retrieval.

**Value Delivery:**  
Uploading a file is only effort, retrieving the right receipt, invoice, or warranty when needed is the moment of value ("aha moment"). Every counted retrieval represents a real user avoiding a 20-minute to 2-hour drawer/WhatsApp hunt, a duplicate supplier payment (`12`), or a missed claim/audit deadline (`03, 04, 15`).

**Long-term Success Indicator:**  
Users who experience fast, successful retrieval build trust in the vault, keep capturing weekly receipts (especially spouses, drivers, and staff), and invite their household members or accountants-driving natural multi-user retention and B2B small-office expansion.

---

## Alternative Metrics Considered

### Alternative 1: Total Registered Users (Sign-ups)

**Description:** Cumulative count of accounts created.

**Why we considered it:** Easy to track and looks good in early demos.

**Why we rejected it:** Pure vanity metric. A user who signs up and never captures or retrieves a document still faces the exact same drawer and WhatsApp chaos.

---

### Alternative 2: Total Documents Uploaded

**Description:** Total count of photos or PDFs uploaded to the system per week.

**Why we considered it:** Measures capture activity and early adoption.

**Why we rejected it:** Uploading unnamed files replicates the exact failure mode in `04` (`IMG_2041`) and `14` (`scan_001`). Volume without verified indexing and successful retrieval does not prove the problem is solved.

---

### Alternative 3: Daily Active Users (DAU)

**Description:** Count of unique users opening the app each day.

**Why we considered it:** Standard consumer engagement metric.

**Why we rejected it:** Document keeping is not a daily social habit for households (`03` searches 1–2×/year; `04` reconciles weekly/monthly). Optimizing for daily opens would encourage noisy notifications rather than calm, reliable retrieval when it matters.

---

## AARRR Alignment

### Where Our NSM Sits in the Funnel

**Primary Stage:** Retention

**Why this stage:**  
While the *first* fast retrieval completes user Activation, **Weekly Successful Document Retrievals** primarily measures ongoing **Retention**-proving users not only uploaded documents once, but actually rely on the vault weeks later when taxes, clinic visits, reimbursements, or warranty lookups occur.

### How NSM Impacts Other Stages

| AARRR Stage | Impact of NSM Growth |
|-------------|---------------------|
| **Acquisition** | Fast audit/claim saves (`03, 04`) drive strong word-of-mouth among Georgian families and small-business accountants |
| **Activation** | Driven by Leading Indicator #1 (capturing & indexing 3+ real documents within 48 hours so there is something to retrieve) |
| **Retention** | This IS our core retention signal-workspaces performing 2+ retrievals/month rarely churn |
| **Referral** | Households invite spouses (`03, 13`) and directors/accountants invite field staff (`04, 12`) to contribute receipts into one shared vault |
| **Revenue** | (N/A for MVP - free beta; future small-office/accounting tier priced on workspace volume and team seats) |

---

## Target & Benchmarks

### Current Baseline

**Current Performance:** 0 (pre-launch MVP; built in Weeks 6–8)

**Measurement Period:** Pre-launch baseline estimate from Lab 5 interviews (`03`–`15`)

**Data Source:** Discovery interview logs (`01-discovery/interview-logs/`) - small-office/admin users (`04, 11, 12, 14, 15`) perform 2–10 document lookups/reconciliations per week, while households (`03, 13`) search every 2–8 weeks.

---

### MVP Target

**Target Value:** 75 Weekly Successful Document Retrievals

**Timeframe:** End of Week 8 (MVP launch + 2 weeks of beta usage)

**Rationale:**  
Realistic for an initial beta of ~25 active users/workspaces (targeting `03`/`04` commitments plus 15–20 household/small-office beta testers from `icp.md`): ~15 small-office/admin users averaging 4 retrievals/exports per week (`60`) + ~10 household users averaging 1.5 lookups per week (`15`) = `75/week`.

---

### Success Criteria

**Minimum Viable Success:** `35` weekly retrievals - Proves at least 10–12 early users consistently find saved documents instead of falling back to WhatsApp/drawers.

**Strong Success:** `75` weekly retrievals - Validates clear product-market fit across early household and small-office testers, ready to scale onboarding.

**Exceptional Success:** `150+` weekly retrievals - Indicates multi-user workspace sharing (staff/spouses contributing and retrieving) is spreading organically.

---

## Measurement Strategy

### How We'll Track This Metric

**Primary Tracking Method:** Product analytics (PostHog Free Tier) paired with backend timestamped event logging (defined in `/02-analytics/event-schema.md`).

**Event(s) That Power This Metric:**
- `document_searched` - When user enters a search query or applies vendor/date/category filters
- `document_retrieved` - Primary NSM event: fired when user opens, downloads, or exports a matching indexed document after a search/filter (capturing `retrieval_time_seconds`)
- `document_indexed` - Prerequisite event verifying metadata (`vendor`, `date`, `amount`, `category`) was extracted

---

### Reporting Cadence

**How often we'll review this metric:**
- [ ] Daily
- [x] Weekly  
- [ ] Bi-weekly
- [ ] Monthly

**Review process:**  
Reviewed every **weekend standup** (per `team-contract.md`) by all four members. Team reviews: (1) current 7-day NSM count vs. 75 target, (2) week-over-week trend, (3) Household vs. Small-Office breakdown, and (4) median `retrieval_time_seconds` (target `< 15s`).

---

### Dashboard & Visualization

**Where the metric lives:** PostHog MVP Dashboard (URL added upon Week 6 setup)

**Visualization format:** Big-number KPI card + 7-day rolling trend line chart

**Additional breakdowns:**
- By user segment (`household` vs. `small_office_admin` vs. `business_owner`)
- By retrieval method (`keyword_search`, `filter_by_category`, `expiry_view`)
- By result action (`opened`, `exported_pdf`, `shared_link`)

---

## Leading Indicators

### Leading Indicator 1: New User Activation Rate

**What it measures:** Percentage of new signups who capture and successfully index at least 3 readable documents within 48 hours of registration.

**Why it predicts NSM:** A vault with 0–1 documents cannot deliver fast retrieval later; users who index 3+ real receipts immediately test search and build the habit.

**Target:** `≥ 60%` activation rate within 48 hours.

---

### Leading Indicator 2: Indexed Metadata Completeness Rate

**What it measures:** Percentage of captured documents that have legible `vendor`, `document_date`, and `category` populated (avoiding `IMG_2041` / `scan_001` dead files).

**Why it predicts NSM:** Searchable metadata is the direct precondition for `< 15s` retrieval; unindexed uploads fail exactly like Google Drive folders (`04`).

**Target:** `≥ 90%` of captured documents cleanly indexed.

---

### Leading Indicator 3: Multi-Contributor Workspace Rate

**What it measures:** Percentage of active workspaces that have 2+ members contributing or viewing documents (e.g., spouse + homeowner `03/13`, or driver/staff + accountant `04/12`).

**Why it predicts NSM:** Pattern #1 showed fragmentation happens across *people*; shared workspaces eliminate month-end WhatsApp chasing (`04`) and double weekly capture volume.

**Target:** `≥ 35%` of active workspaces invite at least 1 collaborator by Week 8.

---

## Relationship to Problem Statement

### Problem Statement (Reference)

**Your validated problem:** Household document keepers and small-organization administrators waste 2–8 hours/month and lose 140–500+ GEL per incident because scattered, unindexed documents cannot be retrieved quickly during urgent claims, filings, and inspections.

**Core user segment:** Household document keepers and small-organization administrators/accountants in Georgia (`03, 04, 11–15`).

**Quantified impact:** 2–8 hours/month wasted searching/chasing; 140–500+ GEL lost per missing document incident.

---

### How NSM Measures Problem Solution

**If our NSM goes up, it means:**
- Users are actually relying on our indexed vault to locate proof in seconds instead of digging through drawers, binders, or WhatsApp histories.
- Uploaded receipts and warranties are legible and properly tagged (no `IMG_2041` failure).
- Both household and small-office keepers experience recurring value across audits, reimbursements, and claims.

**If our NSM stays flat or decreases, it means:**
- Users either stopped capturing incoming documents (capture friction too high-need chat/bot forwarding) OR couldn't find what they uploaded (search/OCR indexing failure).
- Our onboarding didn't get users past the empty-vault hurdle (`Leading Indicator #1` bottleneck).
- Target segment mismatch (e.g., households search too infrequently during a 2-week beta, requiring sharper focus on small-office/accountant users).

---

## Risks & Mitigation

### Risk 1: Gaming or Inflating the Metric with Empty Clicks

**How users (or team testing) could game it:** Team QA tests or accidental repeated clicks on the first document in a list could inflate `document_retrieved`.

**Mitigation strategy:** Filter out internal team IDs (`is_internal_test = false`), deduplicate repeated views of the same `document_id` by the same `user_id` within a 5-minute window, and require `retrieval_time_seconds <= 60` from an intentional search/filter action.

---

### Risk 2: Low Natural Retrieval Frequency in Short Household Betas

**Potential disconnect:** A household user (`03`) might capture 10 warranties cleanly in Week 6 and genuinely love the product, yet not have a broken boiler until 4 months later-making 2-week NSM look artificially flat.

**Mitigation strategy:** Segment the dashboard by `household` vs. `small_office_admin`, focus Week 6–8 beta recruitment heavily on weekly/monthly volume users (`04, 11, 14, 15`), and track Leading Indicator #1 (`3+ indexed documents`) + simulated "15-second retrieval challenge" during onboarding.

---

### Risk 3: Blurry Uploads Failing Readability Later

**Description:** Users upload blurry receipt photos (`04`) that count as "captured" but fail when opened for an audit or warranty claim.

**Mitigation strategy:** Track `Indexed Metadata Completeness Rate` (`Leading Indicator #2`) and prompt immediate recapture if vendor/date/amount extraction confidence is low.

---

## Experimentation & Optimization

### How We'll Improve This Metric

**Hypothesis-driven approach:**  
Whenever NSM trails target during Weeks 6–8, we diagnose the funnel (`document_captured` → `document_indexed` → `document_searched` → `document_retrieved`) and run focused 1-week experiments.

### Planned Experiments (3)

1. **Experiment 1 - "15-Second Retrieval Test" Onboarding Prompt:**
   - **Hypothesis:** If new users are prompted immediately after indexing their 3rd document to search for it by vendor or amount, Activation and Week-2 retention will rise because they feel the "aha moment" right away.
   - **Expected NSM impact:** `+25%` increase in 48-hour activation and `+20%` lift in Week 2 retrievals.

2. **Experiment 2 - Instant Share-Sheet / Chat-Forward Capture:**
   - **Hypothesis:** Allowing spouses or field staff (`04, 12, 13`) to forward a photo from WhatsApp/gallery directly into the workspace without opening a full web form will double weekly indexed documents.
   - **Expected NSM impact:** `+35%` increase in multi-contributor workspaces (`Leading Indicator #3`), driving `+30%` more weekly retrievals by accountants/admins.

3. **Experiment 3 - Monthly "Tax & Warranty Ready" Digest Card:**
   - **Hypothesis:** Sending a weekly/monthly summary of newly added expenses and expiring warranties gives household and office admins a natural trigger to review and export proof.
   - **Expected NSM impact:** `+15–20%` lift in `proof_exported` / `document_retrieved` events during end-of-month windows.

---

## Team Alignment

### How We'll Use This Metric

**In team meetings:** Every weekend standup begins with a 5-minute check of our PostHog NSM dashboard before sprint planning.

**In product decisions:** Any proposed Week 6–8 feature that does not directly increase capture simplicity, indexing accuracy, or retrieval speed is deferred to post-MVP backlog.

**In roadmap planning:** Engineering effort focuses first on eliminating friction in the `document_captured → document_indexed → document_retrieved` chain.

### Accountability

- **Metric owner:** Giorgi Gogichashvili (`@gogicho` - Discovery & Metrics synthesis) with Davit Kutateladze (`@QutaInParis` - Delivery)
- **Implementation owners:** Anri Javakhishvili (`@anrj`) & Nikoloz Tukhashvili (`@Tuxa4Life`)
- **Review cadence:** Weekly during weekend standup
- **Escalation criteria:** If NSM falls below `50%` of weekly target for 2 consecutive weeks (or activation `< 30%`), hold an immediate root-cause session with 3 beta user check-ins before writing new features.

---

## Evolution of the Metric

### When We Might Change Our NSM

**Conditions that would trigger a change:**
- [x] Problem statement pivots away from retrieval toward pure accounting/bookkeeping
- [x] User behavior shows an earlier or stronger "aha moment" (e.g., automated tax export packs)
- [x] Product shifts strictly to B2B small-office teams rather than mixed households + offices

### Post-MVP Evolution

In V2, if small-office compliance and household warranty monitoring both scale, we may evolve the NSM to **"Weekly Verified Proof Resolutions"**-combining fast retrievals, one-click monthly tax export packs (`04`), and proactive warranty/certificate renewals resolved before expiration (`15`).

---

## Validation Checklist

- [x] It directly measures value delivery (not vanity)
- [x] It connects to your validated problem statement
- [x] It's measurable with your event schema
- [x] It's understandable by all team members
- [x] It predicts long-term product success
- [x] It's not easily gamed by users
- [x] All team members agree this is THE most important metric
- [x] You have leading indicators defined
- [x] You have a plan to track and report it
- [x] You know what "good" looks like (targets set)

---

## References

- **Problem Statement:** `/01-discovery/synthesis/final-problem-statement.md`
- **Event Schema:** `/02-analytics/event-schema.md`
- **Analytics Plan:** `/02-analytics/analytics-plan.md`

---

## Sign-off

- Giorgi Gogichashvili (`@gogicho`) - Discovery & Synthesis - 18.11.2026
- Davit Kutateladze (`@QutaInParis`) - [19.11.2026]
-
-