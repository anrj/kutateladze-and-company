# MVP Boundary

Value hypothesis: Household and small-company document keepers who capture scattered receipts, warranties, and invoices into one automatically indexed vault will retrieve valid proof in under 60 seconds during urgent audits, repairs, or inspections instead of losing 140–500+ GEL and hours searching.

## The journey
1. Keeper opens their ProofVault workspace on mobile web or desktop.
2. Keeper snaps a photo or uploads a receipt, warranty, or invoice file.
3. System extracts or saves searchable metadata (vendor, date, category, amount) and stores the document.
4. Keeper searches by keyword or filters by category to locate a needed record.
5. Keeper opens or exports the matching readable proof document within 60 seconds.

## Cut list
| Feature | Reason | Back when |
|---------|--------|-----------|
| WhatsApp / Telegram forwarding bot | Value hypothesis is testable via mobile-web camera/file upload without building messaging webhooks; camera roll/browser upload tests whether keepers will file & search at all (04, 12, 13) | Sprint 2, if Week 7–9 testers fail to upload via mobile web |

| Multi-user workspace invites & role permissions | Single-keeper capture-to-retrieval proves core search speed first; multi-person scatter (03, 04, 13, 14) only matters after solo retrieval works | Sprint 2, once >= 5 solo workspaces hit 3+ indexed docs and 1+ retrieval |

| Expiry / warranty alerts & paid-unpaid invoice flags | Ranked #5 (score 5.33, medium intensity; 03, 11, 15) behind Fragmented Storage & Search Crises (8.33, high); users need indexed proof in the vault before alerts have data | Week 10, once Week 8–9 field vaults hold >= 10 active documents |

| Full OAuth, profile pages, & account settings | 0 of 15 logs asked for profiles; heavy onboarding blocks the first capture action and eats Sprint 1 hours—lightweight email/workspace entry reaches Step 2 immediately | Sprint 2, if beta testers need multi-device account recovery |

| One-click tax audit & warranty claim PDF bundles | Marked as future iteration in patterns-analysis.md (04, 15); opening/exporting the single found document already proves retrieval | Week 11+, once accountants (04) validate single-doc retrieval |

| Google Drive / email auto-sync | Drive folders fail because files stay unindexed like IMG_2041 (04), not because users lack cloud storage; direct capture + indexing tests the root cause | V2, if offices ask to import legacy archives |

| Bilateral shop credit ("nisia") & tutoring ledgers | Ruled out in DECISIONS.md (log 01 has no computer; logs 01 and 06 are social-friction disputes outside ICP) | Never (out of ICP) |

## Step to event
| Step | Event | In schema? |
|------|-------|------------|
| 1 | `user_session_started` | yes |
| 2 | `document_captured` | yes |
| 3 | `document_indexed` | yes |
| 4 | `document_searched` | yes |
| 5 | `document_retrieved` | yes |