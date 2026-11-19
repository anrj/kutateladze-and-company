# Architecture

## System Diagram


[Keeper's Phone / Browser]
   |  (1) Opens workspace, (2) Uploads photo/PDF, (4) Searches, (5) Opens proof
   |-------> [PostHog JS] (UI events: user_session_started, document_searched, document_retrieved)
   v
[React + Vite Web App]  (Hosted on Vercel)
   |  REST / JSON + multipart upload
   v
[Express API (Node/TS)] (Hosted on Render / Railway)
   |--> [Postgres DB (Supabase)]            - stores workspaces and indexed metadata (vendor, date, category, amount)
   |--> [Object Storage (Supabase)]         - stores original receipt/warranty images & PDFs without thermal fade
   |--> [Vision / OCR API (Gemini/OpenAI)]  - extracts vendor, date, amount, and category on upload (manual edit fallback)
   |-->[PostHog Node SDK]                  - fires verified backend state events (document_captured, document_indexed)



## Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| **Frontend** | React + Vite + TypeScript | Already matched to `src/lib/analytics.ts` in `event-schema.md`; fast mobile-web upload & search UI with zero setup friction |
| **Backend / API** | Node.js + Express (TypeScript) | Explicitly planned in `event-schema.md` for backend-verified events and handles multipart file uploads + search queries simply |
| **Database** | PostgreSQL via Supabase *(could change in development)* | Managed Postgres with built-in full-text/ILIKE search on vendor, category, and dates without running a separate search engine |
| **File Storage** | Supabase Storage *(could change in development)* | Lives in the same Supabase project as the DB, avoiding separate AWS S3 IAM/bucket setup in a 9-evening sprint |
| **OCR / Extraction** | Multimodal LLM / Vision API (e.g., Gemini Flash or OpenAI Vision) with manual edit fallback *(could change in development)* | One API call extracts Georgian/English receipt vendor, date, and amount reliably without training or tuning Tesseract |
| **Analytics** | PostHog (`posthog-js` + `posthog-node`) | Required by course Week 7 gate; free tier easily covers ~2,200 beta events/month and tracks UI + backend mutations separately |
| **Hosting / Deploy** | Vercel (frontend) + Render or Railway (Express API) *(could change in development)* | Git-push auto-deploys to public HTTPS URLs on Day 1 so teammates can walk every story on a phone |