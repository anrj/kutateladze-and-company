# Homework 2 · Anri Javakhishvili (@anrj) · Kutateladze & Co.

## 1. Ship report

| What I shipped                                                                                                                                                                                                                                                                                                                                 | Story or rung              | Commit link(s), from my account                                                                                                                                                                                                                                                 | Live at the production URL? | Walked or checked by   |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | ---------------------- |
| Engine (Python, FastAPI, Postgres with pgvector and pg_trgm): upload and index documents, hybrid search, cited answers, endpoints for the whole journey (`/documents`, `/ask`, `/source-opens`), tests including the full upload, index, ask, open journey; reading Word/Excel/PowerPoint 97-2003 in pure Python; Georgian-specific OCR tuning | S2–S5 (server side) | [d32436e](https://github.com/anrj/nuskha/commit/d32436e5c8849f68a0bfcf1ae9543ad6b052abf7), [42569c0](https://github.com/anrj/nuskha/commit/42569c058865a7f982bda2db93cbaeebf365522b), [3bf4ddd](https://github.com/anrj/nuskha/commit/3bf4ddd7910eccc35bbd71f5ed31979ce8433a5e) | yes                         | DK                     |
| Solid 2 + Tailwind web app (Bun, Vite): UploadPanel, DocumentList, AskPanel, AnswerView, typed API client                                                                                                                                                                                                                                      | S2, S4, S5 (screens)       | [c6efc00](https://github.com/anrj/nuskha/commit/c6efc00a4b9b5503fc6eb67763270e222a9f470c)                                                                                                                                                                                       | yes                         | DK                     |
| PostHog wiring: event names in one file (`app/events.json`) read by both engine and web; server events fire after the database commit; browser events fire only after the server confirms                                                                                                                                                      | S6, rung 3                 | [9e08e4d](https://github.com/anrj/nuskha/commit/9e08e4dcc6f90ebaa16a655f223cff2c84654a35)                                                                                                                                                                                       | yes                         | DK                     |

The piece I am least sure works for a real person, and how I will know by Lab 8: Georgian answers on a real office document (scanned or legacy-font files especially), because my tests use fictional samples. I will look at the field-week people's `answer_returned` events in PostHog: whether `citation_count` is above 0, and whether each is followed by a `source_opened` with the same `question_id`.

One thing I did that was not my assigned story: built the Georgian edge (normalization, Mtavruli folding, transliterated queries, detection of pre-Unicode keyboard-layout text by content) and legacy .doc/.xls/.ppt reading, because the target customer's offices still use those files and the journey fails on their first upload without it. [401ec26](https://github.com/anrj/nuskha/commit/401ec261684632b19f5270ab08a18c53635ff6a9), [42569c0](https://github.com/anrj/nuskha/commit/42569c058865a7f982bda2db93cbaeebf365522b)

## 2. Pattern journal, Weeks 5 to 7

I did not write these entries in their own weeks, so I am writing them now.

### Week 5 · One metric that matters

The team set the North Star Metric as Weekly Successful Document Retrievals (`DECISIONS.md`, Wk5, owner GG). In my build lane I made it countable: one event, `source_opened`, fired only after the server has stored the open in `source_opens`, so the PostHog count can be checked against the database. I kept all event names in one file, `app/events.json`, read by both the engine and the web app, so no event can be typed differently on the two sides. [9e08e4d](https://github.com/anrj/nuskha/commit/9e08e4dcc6f90ebaa16a655f223cff2c84654a35), `02-analytics/north-star-metric.md`

### Week 6 · The smallest complete journey

The journey is six steps: open the app, upload a document, it is indexed, ask a question, get an answer with citations, open a cited source. I built it as one vertical slice first and added file formats after it worked. When no LLM key is set or the AI is busy, the app still completes the journey by showing the best matching passage with its citation, so the journey never dead-ends. Outside my lane, I wrote the Sprint 1 change log when the product moved from receipt OCR to Georgian document Q&A. [d32436e](https://github.com/anrj/nuskha/commit/d32436e5c8849f68a0bfcf1ae9543ad6b052abf7), `03-build/sprints/sprint-1.md`, `DECISIONS.md` Wk7 lines (owner AJ)

### Week 7 · Ship to learn

Hack: I wrote `scripts/make_samples.py` to generate fictional Georgian invoices, contracts and HR orders, so I could test the journey without real people's documents, and added an offline embedding option (`EMBED_PROVIDER=ngram`) so development does not depend on an API key. Walking the live journey and reading PostHog showed the server events arriving while my own browser's ad blocker dropped every browser event, so I know the north star count can undercount. I also matched the event schema to the events the code really sends (v1.3). [d32436e](https://github.com/anrj/nuskha/commit/d32436e5c8849f68a0bfcf1ae9543ad6b052abf7), `03-build/analytics/event-schema.md`

## 3. My DECISIONS.md lines

Wk7 · Decided to rework the architecture during and after the sprint ( · evidence: 03-build/architecture/architecture.md · owner: AJ
Evidence: once building started, the planned stack did not fit the realistic scope; `03-build/architecture/architecture.md` records the new stack (Solid 2 web app, FastAPI engine, Neon Postgres, Gemini, R2).
Wk7 · Changes mid sprint dure to a a decision to gravitate towards more general ocr approach than to focus on warranties and receipts, update appended as evidence 03-build/sprints/sprint-1.md · owner: AJ
Evidence: the Sprint Changes Log I appended to `03-build/sprints/sprint-1.md` records how each story changed for the move to Georgian document Q&A.

## 4. My person this week

Student cousin at KIU · I will go to his dorm room and show him the link
