# Homework 2 · Giorgi Gogichashvili (@gogicho) · Kutateladze & Co.

## 1. Ship report

| What I shipped | Story or rung | Commit link(s), from my account | Live at the production URL? | Walked or checked by |
|---|---|---|---|---|
| Completed patterns analysis (frequency/intensity table, 5 Whys) and refined problem statement (`01-discovery/synthesis/`) | Lab 5 / Pre-S1 | [de70db8](https://github.com/anrj/kutateladze-and-company/commit/de70db89576cb03bcbc6a963340138773c1c854e) | n/a (discovery docs) | me, DK |
| Authored North Star Metric, initial PII-free event schema, and PostHog analytics plan (`02-analytics/`) | Week 5–6 / S6 prep | [607584d](https://github.com/anrj/kutateladze-and-company/commit/607584d20b0a8744aa350e6fcad329486e164382) | n/a (analytics spec) | me, AJ |
| Independent phone reviewer walk of full journey (upload → index → ask → open source) with all 5 events verified in PostHog Live Events; synced `03-build/analytics/event-schema.md` (v1.2) to match `app/events.json` | S6 / DoD walk | [23436b1](https://github.com/anrj/kutateladze-and-company/commit/23436b1677e1905f6530515f38bb335dd9428c46) | yes | me (reviewer), AJ |

**The piece I am least sure works for a real person, and how I will know by Lab 8:** Whether a real accountant or office worker immediately understands why a scanned/image-only PDF fails indexing (since OCR for scans moved to Sprint 2) and trusts clicking the `[1]` citation link. By Lab 8, I will check `document_indexed` (`status = failed`, `failure_reason = no_text`) alongside `answer_returned` vs. `source_opened` counts in PostHog Activity for our field-week users.

**One thing I did that was not my assigned story:** Although S4 (`document_searched`) was my Sprint 1 story, the team pivoted mid-sprint to Nuskha's Georgian Q&A built by AJ, so I took over syncing `03-build/analytics/event-schema.md` with `app/events.json` (`document_uploaded`, `document_indexed`, `question_asked`, `answer_returned`, `source_opened`) and executing the independent teammate phone walk so our Definition of Done and analytics tracking match production. [23436b1](https://github.com/anrj/kutateladze-and-company/commit/23436b1677e1905f6530515f38bb335dd9428c46)

## 2. Pattern journal, Weeks 5 to 7

I did not write these entries in their own weeks, so I am writing them now.

### Week 5 · One metric that matters

When writing `north-star-metric.md`, I rejected vanity counts like sign-ups and total uploads because interviews showed users already dump unnamed files like `IMG_2041` into Drive and still lose money. I anchored the metric on fast proof retrieval (`<= 60s`), which translated cleanly into `source_opened` when Nuskha pivoted to cited Q&A. I also banned raw filenames and search text from PostHog payloads (`607584d`).

### Week 6 · The smallest complete journey

With 22 team hours in Sprint 1, we cut accounts, multi-user workspaces, and expiry alerts in `mvp-boundary.md` to protect one vertical slice: open link → upload → index → ask → open cited source. Instead of coding isolated S4 filter UI against fake data, I focused on verifying the real end-to-end search-to-open flow on mobile without login friction.

### Week 7 · Ship to learn

Mid-sprint, we dropped generic receipt-photo tagging to focus on Nuskha's Georgian-specific strengths (pre-Unicode conversion, translit, morphology). Rather than forcing the old `S3` manual-tag form to match `sprint-1.md`, I walked the live journey on a phone, confirmed all five `events.json` events fired in PostHog Live Events, and updated `event-schema.md` to match what is actually live.

## 3. My DECISIONS.md lines

> Wk5 · Exclude 6 unrelated exploratory logs (02, 05, 07–10) and separate informal notebook disputes (01, 06) from the core document-keeping cluster. synthesize 56 sticky notes across households and small offices · evidence: 01-discovery/synthesis/affinity-map.md, 01-discovery/interview-logs/11-office-admin.md, 01-discovery/interview-logs/15-pharmacy-owner.md · owner: GG

The evidence was our 10 real logs plus 5 practice logs in `01-discovery/interview-logs/`, clustered into 56 sticky notes in `01-discovery/synthesis/affinity-map.md` (`de70db8`).

> Wk5 · Select Pattern #1 (Fragmented Storage Across People, Channels, and Devices) as our primary problem over downstream financial loss alone: scattered capture and unindexed files are the upstream software-solvable root cause behind both search crises and 140–500+ GEL losses · evidence: 01-discovery/synthesis/patterns-analysis.md · owner: GG

The evidence was the `8/15` frequency ranking and 5-Whys analysis in `01-discovery/synthesis/patterns-analysis.md` (`de70db8`).

> Wk5 · Set North Star Metric to "Weekly Successful Document Retrievals" (found and opened/exported in <= 60s; Wk8 MVP target: 75/wk) · evidence: 02-analytics/north-star-metric.md · owner: GG

The evidence was the NSM definition and rejected vanity metrics documented in `02-analytics/north-star-metric.md` (`607584d`).

> Wk7 · Sync 03-build/analytics/event-schema.md (v1.2) to match the 5 implemented constants in app/events.json and complete independent phone reviewer walk across all 5 live PostHog events 

The evidence was verifying all five `app/events.json` events live in PostHog Activity during my independent phone walk logged at the bottom of `03-build/sprints/sprint-1.md` and synced into `03-build/analytics/event-schema.md` (`23436b1`).

## 4. My person this week

**The field-week person I am responsible for:** `Relative, accountant` (from `03-build/sprints/sprint-2.md` and `01-discovery/interview-logs/04-relative-accountant.md`) · **how and when the link reaches them:** I will send the production URL via WhatsApp on Tue 24 Nov at 20:00 and walk through 1–2 real non-sensitive `.pdf`/`.xlsx`/`.docx` office files with them on a 15-minute call to watch if they open the cited source page.