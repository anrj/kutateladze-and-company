# Pattern journal · Anri Javakhishvili (@anrj)

Same prompt every week: Where did you apply this pattern this week, where did you exercise agency beyond your assigned lane, and what did you hack together to test something faster than the plan allowed? Link the DECISIONS.md lines you drove.

## Week 1 · Act like an owner, not a role
My accountability in the team contract is Build: repo, code, deployment. I wrote my three problem statements from things I had watched myself: a cousin going straight to AI on easy homework, students waiting on the university laundry machines, and city-funded renovation projects that stalled and were done badly, including on my own house. When the repo diverged from the remote, I merged the remote's signature change into my work instead of overwriting it. `00-foundation/problem-pool.md`, `00-foundation/team-contract.md`

## Week 2 · Evidence over opinion
Helped score the Four Filters on evidence instead of on which idea sounded best, and reopened the farmers problem when new evidence arrived (a hackathon sponsor who has the fridges). Outside my build lane, I wrote the problem hypothesis and interview script v1 and ran the first interview myself with a corner-store seller, so the team had a real conversation to react to. Hack: wrote the quotes in Georgian first and translated them afterwards, so the original wording survived. `00-foundation/four-filters-scorecard.md`, `01-discovery/problem-hypothesis.md`, `01-discovery/interview-script.md`, `01-discovery/interview-logs/01-shopkeeper-corner-store.md`

## Week 3 · Kill your favorite idea
My own renovation problem was the one I liked most, but it scored 0 on workaround and nobody had been seen working around it, so I took it off the shortlist and put in the AI-learning problem I had actually interviewed my cousin about (log 02). After my interview with the seller, I recorded that shop credit was dropped: she raised unpaid and disputed amounts only after we asked, and she has no computer. Hack: re-scored the scorecard right after each new log instead of waiting for a team meeting. `00-foundation/four-filters-scorecard.md`, `01-discovery/interview-logs/01-shopkeeper-corner-store.md`, `01-discovery/interview-logs/02-student-cousin.md`, `DECISIONS.md` line 3

## Week 4 · The prototype is the conversation
There is no prototype yet, so the conversations are the work. Outside my build lane, I logged interview 05 for Nikoloz (a first-year student searching for an apartment), built the competitive landscape from logs 03 and 04 so it lists what interviewees actually use (a folder, a binder, Google Drive, WhatsApp) with their quotes, and brought the outreach tracker up to date with all six interviews and five planned document interviews. Hack: read every teammate's log in one pass to build the landscape table, instead of asking each of them what they heard. `01-discovery/competitive-landscape-seed.md`, `01-discovery/outreach-tracker.md`, `01-discovery/interview-logs/05-student-renter-first-year.md`, `DECISIONS.md` lines 1 and 2

I did not write the Week 5 to 7 entries in their own weeks, so I am writing them now.

## Week 5 · One metric that matters
The team set the North Star Metric as Weekly Successful Document Retrievals (`DECISIONS.md`, Wk5, owner GG). In my build lane I made it countable: one event, `source_opened`, fired only after the server has stored the open in `source_opens`, so the PostHog count can be checked against the database. I kept all event names in one file, `app/events.json`, read by both the engine and the web app, so no event can be typed differently on the two sides. [9e08e4d](https://github.com/anrj/nuskha/commit/9e08e4dcc6f90ebaa16a655f223cff2c84654a35), `02-analytics/north-star-metric.md`

## Week 6 · The smallest complete journey
The journey is six steps: open the app, upload a document, it is indexed, ask a question, get an answer with citations, open a cited source. I built it as one vertical slice first and added file formats after it worked. When no LLM key is set or the AI is busy, the app still completes the journey by showing the best matching passage with its citation, so the journey never dead-ends. Outside my lane, I wrote the Sprint 1 change log when the product moved from receipt OCR to Georgian document Q&A. [d32436e](https://github.com/anrj/nuskha/commit/d32436e5c8849f68a0bfcf1ae9543ad6b052abf7), `03-build/sprints/sprint-1.md`, `DECISIONS.md` Wk7 lines (owner AJ)

## Week 7 · Ship to learn
Hack: I wrote `scripts/make_samples.py` to generate fictional Georgian invoices, contracts and HR orders, so I could test the journey without real people's documents, and added an offline embedding option (`EMBED_PROVIDER=ngram`) so development does not depend on an API key. Walking the live journey and reading PostHog showed the server events arriving while my own browser's ad blocker dropped every browser event, so I know the north star count can undercount. I also matched the event schema to the events the code really sends (v1.3). [d32436e](https://github.com/anrj/nuskha/commit/d32436e5c8849f68a0bfcf1ae9543ad6b052abf7), `03-build/analytics/event-schema.md`
