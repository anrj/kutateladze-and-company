## Sprint 1 Active Events & Ownership (Lab 6 Reconciliation)

All 5 steps of `03-build/mvp-boundary.md` map 1:1 to existing schema events (zero gaps). Events outside the Sprint 1 journey are deferred to Sprint 2 so every active Sprint 1 event fires in PostHog before the Week 7 gate.

| Step / AARRR | Event | Fired By | Key Properties | Owner | Sprint 1 Status |
|--------------|-------|----------|----------------|-------|-----------------|
| Step 1 · Retention/Entry | `user_session_started` | Frontend (`posthog-js`) | `workspace_id`, `days_since_signup`, `days_since_last_visit`, `workspace_doc_count`, `user_segment` | DK | Active (wire Wk 6) |
| Step 2 · Capture | `document_captured` | Backend (`posthog-node`) | `workspace_id`, `document_id`, `capture_channel`, `file_type`, `contributor_role` | NT | Active (wire Wk 6) |
| Step 3 · Activation | `document_indexed` | Backend (`posthog-node`) | `workspace_id`, `document_id`, `document_category`, `indexing_method`, `has_vendor`, `has_document_date`, `has_amount`, `has_expiry_date`, `metadata_complete` | AJ | Active (wire Wk 6) |
| Step 4 · Lookup | `document_searched` | Frontend (`posthog-js`) | `workspace_id`, `search_id`, `search_mode`, `query_length_chars`, `results_count` | GG | Active (wire Wk 6) |
| Step 5 · North Star | `document_retrieved` | Frontend (`posthog-js`) | `workspace_id`, `document_id`, `search_id`, `document_category`, `result_action`, `retrieval_time_seconds`, `document_age_days` | DK | Active (wire Wk 6) |
| Acquisition (cut) | `user_signup_started` | — | Deferred with full OAuth/onboarding cut in `mvp-boundary.md` | — | Deferred to Sprint 2 |
| Acquisition (cut) | `user_signup_completed` | — | Deferred with full OAuth/onboarding cut in `mvp-boundary.md` | — | Deferred to Sprint 2 |
| Referral (cut) | `workspace_invite_sent` | — | Deferred with multi-user workspace invites cut in `mvp-boundary.md` | — | Deferred to Sprint 2 |