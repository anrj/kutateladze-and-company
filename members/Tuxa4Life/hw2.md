# Homework 2 · Nikoloz Tukhashvili (@Tuxa4Life) · Team Kutateladze & Co.

## 1. Ship report

| What I shipped | Story or rung | Commit | Live at the production URL? | Walked or checked by |
|---|---|---|---|---|
| Deployed the app on Cloudflare at https://nuskha.tuxa.ge/ | Sprint 2 story 1 / rungs 1 and 2 | 13d3f6c | yes | me |
| Uploads get indexed in the background, so big files don't hang the upload | Journey: upload and index | b36be8d | yes | me |
| Source links are signed and expire, so they don't leak the user's id | Journey: open source (north star) | 4db585e | yes | me |
| Deleting documents | Journey support | 4b1abec | yes | me |
| Faster keyword search | Journey: ask | 8e0a509 | yes | me |

**The piece I am least sure works for a real person, and how I will know by Lab 8:** background indexing on real office files. I only tested it with our sample documents. I will compare `document_uploaded` and `document_indexed` counts in PostHog and check how many uploads failed.

**One thing I did that was not my assigned story:** the deploy was AJ's story, but the app was still only running locally, so I put it on Cloudflare under my domain so we had a real link to test and send to people.

## 2. Pattern journal, Weeks 5 to 7

I did not write these entries in their own weeks, so I am writing them now.

### Week 5 · One metric that matters

Our north star is `source_opened`, when someone opens the source of an answer. The link for that carried the user's id, which is the only thing protecting their documents. I changed it to a signed link that expires, so people can safely click the thing we want them to click.

### Week 6 · The smallest complete journey

I looked at where someone could get stuck before finishing the journey. Large uploads kept the request open until indexing was done, so I moved indexing to the background. I also added deleting documents, since otherwise there was no way to remove a wrong upload.

### Week 7 · Ship to learn

The app only ran locally, so instead of waiting for the planned setup I deployed it on Cloudflare at nuskha.tuxa.ge. That let us test the full journey on a phone with a real link, and showed us problems we could not see locally.

## 3. My DECISIONS.md lines

> Wk7 · Deploy the app on Cloudflare at https://nuskha.tuxa.ge/ so the journey runs at a public production URL (gate rungs 1 and 2) · evidence: https://github.com/anrj/nuskha/commit/13d3f6cd381422c0dbecc84a1143fea08e241b5d, 03-build/sprints/sprint-2.md · owner: NT

The evidence is the live URL in the Nuskha README and the gate table in `sprint-2.md`.

## 4. My person this week

**The field-week person I am responsible for:** Office admin (log 11) · **how and when the link reaches them:** I will send them https://nuskha.tuxa.ge/ on WhatsApp on Wed 25 Nov.
