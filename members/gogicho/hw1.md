# Homework 1 · Giorgi Gogichashvili (@gogicho) · Team Kutateladze & Co.


## 1. My interview log

Every real interview you took part in. At least 2 where you were the **asker**.

| Log file (link) | My role (asked / logged / both) | What this interview changed, in one sentence |
|[`01-discovery/interview-logs/07-daily-commuter.md`](../../01-discovery/interview-logs/07-daily-commuter.md)|asked |realized intercity travelers care less about the ticket price and more about guaranteed departure times, because arriving exhausted defeats the purpose of their trip.|
|[`01-discovery/interview-logs/08-office-coworker.md`](../../01-discovery/interview-logs/08-office-coworker.md)|both |we both learned that coworkers aren't skipping lunch to save time, they are skipping it because delivery fees for a single person make nearby food overly expensive. |
|[`01-discovery/interview-logs/09-remote-worker.md`](../../01-discovery/interview-logs/09-remote-worker.md) |logged | discovered the biggest cost of remote overseas work isn't just sleeplessness, but the unpredictability of late-night meetings ruining family relationships. |

**The quote I heard myself that mattered most, with its log number:**

> ""I spent 40 GEL on a mid, super greasy and disgusting burger just so that my stomach wouldn't aggressively make noises during the 2 PM client meeting, and I hated myself for it." — 08-office-coworker.md"

**One moment I broke the Mom Test, and what I asked instead next time** (name the log):

In 07-daily-commuter.md, I broke the Mom Test by pitching a solution, I asked: "Would an app that shows you exactly how many seats are left in the marshrutka help you avoid being late?" The interviewee just said "Yeah, probably," which gave me false validation. However, next time in 08-office-coworker.md, I asked about past behavior instead: "Walk me through exactly what you did for lunch yesterday, from the moment you realized you were hungry.", which revealed the reality and the exact financial cost they paid.

## 2. Pattern journal, Weeks 1 to 4

Copy your entries from `members/<your-github-username>/pattern-journal.md`. Each week answers the same prompt: Where did you apply this pattern this week, where did you exercise agency beyond your assigned lane, and what did you hack together to test something faster than the plan allowed? Each entry links at least one file or commit.

### Week 1 · Act like an owner, not a role

Where applied: We were trying to decide which of the three problems (intercity transit, workplace lunch, or remote timezones) to pursue for our product. More potential problems added as other group members wrote up their problems. We were stuck waiting on a formal market analysis.

Agency beyond my lane: Haven't really stepped out beyond my lane.

What I hacked together: Instead of a complex survey, I just posted a quick poll in our actual workplace whatsapp asking coworkers how much they spent on lunch that week. It proved the pain point in 20 minutes.

### Week 2 · Evidence over opinion

Where applied: Our team had a huge debate over whether our lunch solution(in case this problem was picked) should focus on "healthy food" or just "cheap food." Half the team insisted people wanted salads, the other half said people just wanted to save money.

Agency beyond my lane: Stayed in my lane still.

What I hacked together: I created a fake menu in a Google Form with two options: a $10 healthy salad and a $5 basic sandwich. I sent it to 10 coworkers saying I was picking up food. 8 out of 10 chose the $5 sandwich. Evidence killed the "healthy food" opinion instantly.

### Week 3 · Kill your favorite idea

Where applied: I was originally very passionate about solving the remote-worker timezone problem by building an automated Google Calendar script that reschedules meetings based on circadian rhythms.

Agency beyond my lane: I looked at our team's workflow and realized the calendar integration would take a month to build and require massive API permissions.

What I hacked together: I officially killed my timezone-scheduler idea so the team could focus 100% on the other idea.

### Week 4 · The prototype is the conversation

Where applied: We needed to know if users would actually trust an AI to summarize their lengthy printed documents, or if they feared it would miss important details and ignore the feature entirely.

Agency beyond my lane: Stayed in my lane.

What I hacked together: I acted as the AI myself. I told 5 coworkers to just send me photos of long documents on WhatsApp whenever they didn't want to read them. When I got a photo, I manually typed the text into ChatGPT, generated the summary, and texted it back to them within minutes. They absolutely loved it and immediately started making decisions based on the summaries, proving they trust the concept without us writing a single line of AI code.

## 3. My DECISIONS.md citations

Each `DECISIONS.md` line you drove or supplied the evidence for. Copy the line exactly, then say what you did.

| The line, copied | What I did for it |
|Coworkers skipping lunch or paying high single-person delivery fees due to lack of a cafeteria · evidence: 00-foundation/problem-pool.md (AJ, Team), 00-foundation/four-filters-scorecard.md · runner-up: Intercity transit unpredictability · owner: GG|I evaluated our three initial problem spaces (transit, remote work, lunch) using the Four Filters scorecard. I realized the lunch problem was the most actionable for our team's skills and drove the decision to make it our primary focus, leaving transit as the runner-up. However, teams other ideas triumphed in the end. |
|"We are wrong if the people we interview do not raise the problem or its cost on their own, when they describe the hardest part of the task, before we mention it." verdict: confirmed · evidence: 01-discovery/interview-logs/log-08-office-coworker.md, 01-discovery/interview-logs/log-10-office-coworker.md · owner: GG | I conducted the interviews with Coworker-A and Coworker-B (Logs 08 & 10). By asking open-ended questions about their past behavior ("walk me through your lunch yesterday"), I successfully triggered the prediction when they independently brought up the 40 GEL financial cost and the pain of starving.  |
| | |

---

**Before you upload, check**

- [x] 2 or more rows say "asked" or "both", and each links a real log in `01-discovery/interview-logs/`.
- [x] Four journal entries, each with a link. The commit history shows they were written across the weeks, not tonight.
- [x] Every DECISIONS.md line is copied word for word, and the owner or evidence on it connects to you.
- [x] Committed from your own GitHub account. Uploaded to the LMS before the deadline.