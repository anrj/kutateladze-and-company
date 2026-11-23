# Architecture

## System Diagram

```
[Office worker's browser / phone]
   |  (2) Uploads document, (4) Asks a question, (5) Opens the cited source
   |-------> [PostHog JS]  UI events after the server confirms: document_searched, document_retrieved
   v
[Solid 2 + Vite web app]  (hosted on Vercel / Cloudflare Pages)
   |  REST / JSON + multipart upload, anonymous X-User-Id header
   v
[FastAPI engine (Python)]  (hosted on Render)
   |--> Parse          PDF, Word/Excel/PowerPoint (incl. 97–2003), ODT, RTF, CSV, HTML, TXT
   |--> Georgian layer normalization, Mtavruli folding, pre-Unicode text → Unicode, translit queries
   |--> [Gemini API]   embeddings (gemini-embedding-001) + cited answers (Gemini Flash, with fallback models)
   |--> [Postgres (Neon)]  documents, text chunks + vectors (pgvector), keyword index (pg_trgm),
   |                       questions, source_opens (North Star rows)
   |--> [Object storage]   original files: local disk in dev, Cloudflare R2 (S3 API) in production
   |--> [PostHog Python SDK] backend events after the DB commit: document_captured, document_indexed
```

### Request flow

1. **Upload:** file stored → document row committed (`document_captured`) → text extracted per page/slide/sheet → Georgian normalization → chunked (~800 chars, page kept) → embedded → chunks saved → status committed (`document_indexed`).
2. **Ask:** query variants (original + Latin→Georgian translit) → hybrid search per variant (pgvector similarity + trigram keyword match) → fused with Reciprocal Rank Fusion → top 6 chunks → Gemini writes an answer citing [1], [2]… → question row saved → browser fires `document_searched`.
3. **Open source:** browser asks the engine to record the open → `source_opens` row committed → browser fires `document_retrieved` → the file opens at the cited page.

If every Gemini model is busy, the engine still returns the best matching passage with its citation, so the journey never breaks.

## Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| **Frontend** | Solid 2 + Vite + TypeScript + Tailwind 4, built with Bun | Small, fast UI (~65 KB JS); signals model; Tailwind instead of a component library because none supports Solid 2 yet |
| **Backend / API** | Python + FastAPI (managed with uv) | Python has the best document parsing and text tooling; FastAPI gives typed endpoints and OpenAPI for free |
| **Database** | PostgreSQL on Neon with `pgvector` + `pg_trgm` | One managed database for records, vector search and keyword search; no separate search engine. Trigram matching handles Georgian word endings, which Postgres full-text search can't stem |
| **File storage** | Local disk (dev) / Cloudflare R2 (prod) behind one interface | R2 is S3-compatible with a free tier and no egress fees; the same interface keeps files local in a future desktop version |
| **AI (embeddings + answers)** | Google Gemini API (free tier) via its OpenAI-compatible endpoint | Good Georgian quality at no cost for the demo; any OpenAI-compatible provider (Groq, OpenRouter, Mistral) can be swapped in through env vars |
| **Document parsing** | PyMuPDF, python-docx, openpyxl, python-pptx, xlrd, olefile, striprtf (+ LibreOffice if installed) | Covers the formats Georgian offices actually use, including legacy .doc/.xls/.ppt, in pure Python so it runs on free hosting |
| **Analytics** | PostHog EU Cloud (`posthog-js` + `posthog-python`) | Required by the Week 7 gate; browser events fire after server confirmation, backend events after the DB commit |
| **Hosting / Deploy** | Vercel or Cloudflare Pages (web) + Render (engine, `render.yaml`) | Git-push deploys to public HTTPS URLs; free tiers cover the beta |

## Key Decisions

- **Own lightweight RAG pipeline instead of a framework** (LangChain, RAGFlow): the Georgian-specific steps (pre-Unicode detection, translit, trigram matching) sit inside the pipeline, and a framework would make them harder to control. The core loop is a few hundred lines.
- **Hybrid search** (vector + keyword): embeddings find meaning, trigram matching catches exact names, TINs (ს/კ), invoice numbers and Georgian word forms.
- **No accounts in the MVP:** an anonymous per-browser ID scopes documents and doubles as the PostHog `distinct_id`.
- **Pre-Unicode Georgian is detected by content, not font names:** works for every format, including CSV and TXT.
- **Local-first later:** storage and embeddings sit behind interfaces, so the same engine can run on the user's machine in a desktop version.

## Repository Layout

```
app/
  events.json        analytics event names, shared by engine and web
  engine/            FastAPI engine (nuskha/: georgian, ingest, embed, search, answer, api)
  web/               Solid 2 web app
render.yaml          engine deploy blueprint
.env.example         engine settings (no secrets)
```