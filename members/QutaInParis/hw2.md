# Homework 2 · Davit Kutateladze (@QutaInParis) · Kutateladze & Co.

## 1. Ship report

| What I shipped | Story or rung | Commit link(s), from my account | Live at the production URL? | Walked or checked by |
|---|---|---|---|---|
| Drafted Sprint 1 MVP boundary (`03-build/mvp-boundary.md`), architecture (`03-build/architecture/architecture.md`), and Sprint 1 plan (`03-build/sprints/sprint-1.md`) | Lab 6 / Pre-S1 | [1e1471045484d9b45ea3d7b05c503c877471d4c5](https://github.com/anrj/kutateladze-and-company/commit/1e1471045484d9b45ea3d7b05c503c877471d4c5) | n/a (planning docs) | me, GG |
| Executed Story 6 production phone walk on `https://nuskha.tuxa.ge/` with mobile data (Wi-Fi off) while NT read PostHog Activity aloud; logged Story 3, 5, and 7 reviewer checks and field-week outreach in `03-build/sprints/sprint-2.md` | Sprint 2 / Stories 3, 5, 6, 7 | [d719273dfa1f4f7f69f8cffa6992e5f4f9ff05dc](https://github.com/anrj/kutateladze-and-company/commit/d719273dfa1f4f7f69f8cffa6992e5f4f9ff05dc) | yes | me, NT, AJ, GG |
| Updated live production verification status for all five active Nuskha events (`document_uploaded`, `document_indexed`, `question_asked`, `answer_returned`, `source_opened`) in `03-build/analytics/event-schema.md` | Sprint 2 / Story 6, Rung 3 | [1fe71edc108ddba9a456bfed16bf5f2adcbe70e4](https://github.com/anrj/kutateladze-and-company/commit/1fe71edc108ddba9a456bfed16bf5f2adcbe70e4) | yes | me, NT |

**The piece I am least sure works for a real person, and how I will know by Lab 8:** Whether a non-technical office admin immediately realizes they must upload text-based PDFs or Word/Excel files rather than phone photos or scanned PDFs (since OCR moved to Sprint 2), and whether they trust clicking the inline citation to verify the source page. By Lab 8, I will check my field-week person's feedback alongside `document_indexed` failure counts and `answer_returned` vs. `source_opened` conversion in PostHog.

**One thing I did that was not my assigned story:** After our mid-sprint pivot cut `S1` (workspace accounts) and folded `S5` into Nuskha's `source_opened` flow built by AJ, I took over verifying the Sprint 2 gate artifacts across `app/`, `.env.example`, `README.md`, and the public PostHog dashboard in an incognito browser, logging the checks in `03-build/sprints/sprint-2.md` ([HASH_2](https://github.com/anrj/kutateladze-and-company/commit/FULL_HASH_2)).

## 2. Pattern journal, Weeks 5 to 7

I did not write these entries in their own weeks, so I am writing them now.

### Week 5 · One metric that matters

While refining Problem Statement v2 (`01-discovery/synthesis/final-problem-statement.md`), logs `03` and `04` showed people already had folders and Google Drive yet still lost hundreds of lari when they could not pull up valid proof fast. That convinced me to anchor our North Star on retrieval speed (`<= 60s`) rather than total uploads. Even after pivoting to Nuskha, the metric stayed focused on opening the cited source (`source_opened`), not counting uploaded files.

### Week 6 · The smallest complete journey

In `03-build/mvp-boundary.md`, I cut accounts, multi-user invites, WhatsApp bots, and expiry alerts so the team could ship one vertical journey inside our Sprint 1 hours. When we realized mid-sprint that even named workspaces (`S1`) slowed users down before their first upload, we cut accounts entirely in favor of an anonymous browser ID so users land straight on Upload and Ask.

### Week 7 · Ship to learn

Because we did not have a clickable Prototype v1 entering Lab 6, I logged the catch-up track in `DECISIONS.md` so we could test real users directly on the deployed production journey instead of building throwaway mockups. Walking `https://nuskha.tuxa.ge/` on mobile data while NT verified each live event in PostHog surfaced real mobile and file-format edge cases faster than rewriting specs.

## 3. My DECISIONS.md lines

> Wk5 · Problem statement v2 (decision: REFINE, confidence: Medium): people already have folders, binders, and Google Drive, so losses (140–500+ GEL, 2–8 hrs/mo) stem from multi-person/multi-channel capture without searchable metadata during urgent audits, boiler failures, warranty claims, and inspections; · evidence: 01-discovery/synthesis/final-problem-statement.md, 00-foundation/icp.md · owner: DK

The evidence was our refined synthesis across logs `03, 04, 11–15` in `01-discovery/synthesis/patterns-analysis.md` and the user profiles in `00-foundation/icp.md`.

> Wk6 · Lock MVP boundary to 5-step solo keeper journey (open workspace -> capture -> index -> search -> retrieve in <= 60s); cut WhatsApp bot, multi-user invites, expiry alerts, full OAuth/settings, Drive sync, and tax bundles · evidence: 03-build/mvp-boundary.md, 01-discovery/synthesis/patterns-analysis.md · owner: DK

The evidence was the Pattern #1 rank score (`8.33`) versus secondary features in `01-discovery/synthesis/patterns-analysis.md`, documented in `03-build/mvp-boundary.md`.

> Wk6 · Catch-up track: Prototype v1 not built yet, so Sprint 1 runs 3 user-testing sessions on the first deployed S1–S5 journey screen (or quick clickable v1) and logs findings before Week 7 lab · evidence: 03-build/sprints/sprint-1.md · owner: DK

The evidence was our pre-Lab-6 prototype status and the Sprint 1 catch-up testing plan recorded in `03-build/sprints/sprint-1.md`.

## 4. My person this week

**The field-week person I am responsible for:** `Office admin` (from `03-build/sprints/sprint-2.md` and `01-discovery/interview-logs/11-office-admin.md`) · **how and when the link reaches them:** I will send `https://nuskha.tuxa.ge/` via WhatsApp on 2026-11-25 and follow up by call the next day to see if they uploaded a real non-sensitive office document and opened a cited source.