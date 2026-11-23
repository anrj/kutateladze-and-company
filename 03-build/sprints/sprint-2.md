# Sprint 2: the field week

Sprint 2 runs from Lab 7 to Lab 8. Real people use the product this week, and Lab 8 opens by reading what they did.

## Sprint 1 review

Judged against the Sprint 1 definition of done: merged to `main`, deployed at the public URL, walked on a phone by a teammate who did not build it, and its events show in the PostHog live view.

| Story | Owner | Done by our definition? | If not: which clause is missing, and what happened |
|---|---|---|---|
| S6 Deploy stack and PostHog pipeline | AJ | No | "Deployed at the public URL" and "events show in PostHog live view": Neon is connected and the PostHog project exists, but the app only runs locally. "Merged to `main`": the app code is not yet in this repo's `app/` folder. |
| S1 Workspace entry | DK | No (cut) | Cut by decision on 24.11: no accounts or workspaces in the MVP. An anonymous per-browser ID is the PostHog `distinct_id`, so `user_session_started` was dropped. |
| S2 Capture document | NT | No | Works locally. Missing: deployed, walked on a phone, event shown in PostHog (event still named in code as `document_uploaded`; schema v1.1 name is `document_captured`). |
| S3 Index metadata | AJ | No | Works locally with text extraction, Georgian normalization and embeddings. Missing: deployed, walked, event in PostHog. OCR field extraction moved out by the slip rule. |
| S4 Search and filter | GG | No | Works locally as question to cited answer. Missing: deployed, walked, event in PostHog (code names `question_asked` and `answer_returned`; schema v1.1 has one event, `document_searched`). |
| S5 Retrieve proof (north star) | DK | No | Works locally. Missing: deployed, walked, event in PostHog (code names it `source_opened`; schema v1.1 name is `document_retrieved`, plus `retrieval_time_seconds`). |

Slip rule: fired. S3 automated OCR extraction moved to Sprint 2. S4 and S5 stayed in Sprint 1, as written. The product also changed scope to Georgian document question answering (see the Sprint Changes Log in `sprint-1.md`).

## The gate

| Rung | Reached? | Evidence: a link or a number | If not: owner, and by when |
|---|---|---|---|
| 1 Production URL loads on a phone | No | App runs locally only (Sprint 1 change log) | AJ, by Tue 24 Nov 22:00 |
| 2 Journey completes there | No | n/a | AJ, by Tue 24 Nov 22:00 |
| 3 North star event in Activity, from production | No | n/a | AJ, by Wed 25 Nov 12:00 |
| 4 Counts reconcile | No | n/a | GG, by Wed 25 Nov 18:00 |
| 5 Dashboard, linked in README | No | n/a | GG, by Wed 25 Nov 18:00 |

## Real hours before Lab 8

| Initials | Hours free | When, specifically |
|---|---|---|
| DK | [ ] | [ ] |
| AJ | [ ] | [ ] |
| GG | 4 | [ ] |
| NT | [ ] | [ ] |
| Total | [ ] | |

## Stories this week

In this order: missing rungs, then anything that slipped from Sprint 1. No new story: no walk or test session has asked for one yet.

1. Deploy engine (Render, files on R2) and web app (Vercel or Cloudflare Pages) so the journey upload, index, ask, open cited source completes at the production URL on a phone, no console or CORS errors (rungs 1 and 2) · owner AJ · reviewer GG · 3 h
2. Align event names and properties with schema v1.1 in `app/events.json` (`document_captured`, `document_indexed`, `document_searched`, `document_retrieved`, with `search_id`, `retrieval_time_seconds` and indexing `status`), and see each fire once from production in PostHog Activity (rung 3) · owner AJ · reviewer NT · 2 h
3. Put the app code in this repo's `app/` folder, with `.env.example` and no secrets in history; fill the README live-product row · owner AJ · reviewer DK · 1 h
4. Reconcile: PostHog `document_retrieved` count equals the `source_opens` rows in Neon over the same window, both numbers written under the gate block in `event-schema.md`; internal-user filter on (rung 4) · owner GG · reviewer AJ · 1.5 h
5. Dashboard with three tiles (north star trend, journey steps, retention), internal filter on each, shared no-login link in the README (rung 5) · owner GG · reviewer DK · 2 h
6. Phone walk: a teammate who did not build it does the journey on the production URL with Wi-Fi off, while a second teammate reads each event row in PostHog Activity aloud; Live column filled in the schema · owner DK · reviewer NT · 1 h
7. Slipped from Sprint 1: three test sessions logged in `02-design/user-testing/usability-findings.md` on the deployed journey (catch-up track in DECISIONS.md, Wk6) · owner NT · reviewer DK · 3 h

Story hours total 13.5 h. They must not exceed the total in the hours table above.

Not this week: OCR for photos and scanned PDFs, accounts and workspaces, category filters, result reranking (all moved to Sprint 2 backlog in the Sprint 1 change log, but not scheduled until the gate is passed).

## Field week: who uses it

3 to 5 real people from our ICP, not classmates, code names and roles only. Each gets the production URL, never a preview link.

| Code name and role | Log or session they come from | How the link reaches them | Sent by, and by when | Opened? (fill in during the week) |
|---|---|---|---|---|
| Office admin | `01-discovery/interview-logs/11-office-admin.md` | [ ] | [ ] | |
| Small-company director | `01-discovery/interview-logs/12-small-company-director.md` | [ ] | [ ] | |
| University admin | `01-discovery/interview-logs/14-university-admin.md` | [ ] | [ ] | |
| Relative, accountant | `01-discovery/interview-logs/04-relative-accountant.md` | [ ] | [ ] | |

What we tell them: "We are students building a tool that lets you upload your Georgian office documents and ask a question to get an answer that shows the exact file and page. Please try it with one or two real documents; we count how many people open the source of an answer, and we do not see your document names or questions."

## Prediction

Written in the lab, before the week starts. Never edited afterwards; Lab 8 reads it as written.

By Lab 8: 12 `source_opened` events from real people, not our team. Also: 4 people reach `document_indexed` with status `ready`.

We are wrong if: fewer than 4 `source_opened` events happen across the entire week, or more than `50%` of real uploads hit `document_indexed` with `status = failed` (because users mostly upload scanned/image PDFs without text).

Mid-week look: GG opens the dashboard on thursday and writes one line here: what the counts are and one thing we change because of them, or nothing.

## The DECISIONS.md line

Add this to the bottom of `DECISIONS.md` (replace it with the "Gate passed" form from the template when it closes, linking the old line):

Wk7 · Gate open at rung 1: missing public deploy of engine and web app, event rename to schema v1.1, reconcile and dashboard, owners and times in sprint-2.md · evidence: 03-build/sprints/sprint-2.md · owner: AJ