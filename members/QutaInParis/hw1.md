# Homework 1 · Davit Kutateladze (@QutaInParis) · Team Kutateladze & Co.

## 1. My interview log

| Log file (link) | My role (asked / logged / both) | What this interview changed, in one sentence |
|---|---|---|
| [`01-discovery/interview-logs/03-house-owner.md`](../../01-discovery/interview-logs/03-house-owner.md) | asked | It proved that people pay significant out-of-pocket costs (220 GEL) for covered repairs due to lost receipts, validating that paper clutter carries an immediate financial penalty rather than just minor friction. |
| [`01-discovery/interview-logs/04-relative-accountant.md`](../../01-discovery/interview-logs/04-relative-accountant.md) | asked | It revealed that manual Google Drive workarounds fail because people don't name files, expanding our problem scope to small business expense chaos where missing source slips result in actual tax fines (500 GEL). |

**The quote I heard myself that mattered most, with its log number:**

> "ცენტრალური გათბობის ქვაბი რომ გაითიშა შუა ზამთარში, ვიცოდი რომ საბუთი სადღაც მქონდა, მაგრამ ვერსად ვერ ვიპოვეთ. ბავშვები იყინებოდნენ, ამიტომ გამოვიძახეთ სხვა ხელოსანი და 220 ლარი გადავუხადეთ უცებ" / "When the central heating boiler shut down in the middle of winter, I knew I had the document somewhere, but we couldn't find it anywhere. The kids were freezing, so we just called another repairman and paid 220 GEL right away." (01)

**One moment I broke the Mom Test, and what I asked instead next time** (name the log): In log 01 I caught myself pitching the solution early by asking: "Would you use an app that keeps all your warranties in one place so you never lose them?", which invited an unhelpful, polite "yes." In log 02 I stopped asking about future intentions and asked about past behavior instead: "Walk me through what happened the last time you had to submit expense documents for an audit, and where you found them" in `01-discovery/interview-script.md`.

## 2. Pattern journal, Weeks 1 to 4

Copy your entries from `members/QutaInParis/pattern-journal.md`. Each week answers the same prompt: Where did you apply this pattern this week, where did you exercise agency beyond your assigned lane, and what did you hack together to test something faster than the plan allowed? Each entry links at least one file or commit.

### Week 1 · Act like an owner, not a role
My primary accountability in the team contract is Product & Customer Discovery. Rather than waiting for a finalized project scope, I sourced 3 concrete problem statements directly from acute friction I observed: lost appliance warranties costing repair money, thermal ink fading on purchase slips, and office document disorganization. When our GitHub repo was missing structure for discovery deliverables, I set up the directory folders and template files myself so teammates could immediately start logging notes without waiting for the build lead. `00-foundation/problem-pool.md`, `00-foundation/team-contract.md`

### Week 2 · Evidence over opinion
Our team debated whether to target students managing study notes or households managing paperwork. I pushed to score the Four Filters strictly using concrete user evidence rather than personal preference. Stepping outside my discovery lane, I set up the interview logging structure, drafted the script based on Mom Test principles, and conducted the first two interviews myself (homeowner and accountant) to bring back hard numbers: a 220 GEL avoidable repair bill and a 500 GEL tax penalty. Hack: conducted the interviews on short notice over phone/in-person, jotted down verbatim Georgian quotes immediately, and translated them right away to preserve exact emotional language. `00-foundation/four-filters-scorecard.md`, `01-discovery/interview-logs/03-house-owner.md`, `01-discovery/interview-logs/04-relative-accountant.md`

### Week 3 · Kill your favorite idea
I initially championed a complex desktop web portal where users would manually categorize, tag, and cross-reference documents with custom folder trees. The evidence killed this idea: the accountant showed me that employees already refuse to rename files on Google Drive, and the homeowner stores papers in shoe boxes precisely because nobody wants to manually type tags. I abandoned the desktop folder architecture and redirected our team to focus purely on rapid mobile photo capture with automatic OCR extraction. `01-discovery/problem-hypothesis.md`, `DECISIONS.md`

### Week 4 · The prototype is the conversation
Instead of waiting for a fully functional web frontend or database backend, I hacked together a basic Telegram bot linked to a lightweight OCR script to simulate automated receipt logging. I took this artifact back to the accountant and had her forward three crumpled test receipts into the chat. Watching her react to the bot parsing the store name, date, and total amount in real time gave us immediate feedback on extraction errors that a 20-minute abstract conversation never could have surfaced. `02-prototype/bot-prototype.py`, `02-prototype/feedback-log.md`

## 3. My DECISIONS.md citations

Each `DECISIONS.md` line you drove or supplied the evidence for. Copy the line exactly, then say what you did.

| The line, copied | What I did for it |
|---|---|
| `Decision: Pivot target product from desktop web organizer to mobile-first instant photo capture (Owner: Davit Kutateladze)` | I conducted Interview 01 and 02, bringing evidence that neither homeowners nor employees sit at computers to file papers, making mobile photo capture the only workflow they will realistically use. |
| `Decision: Require automated receipt parsing (OCR) in MVP scope rather than manual entry (Evidence: Interview 02)` | I logged Interview 02 where the accountant demonstrated that manual Google Drive tagging failed entirely due to user friction, proving manual entry would kill retention. |