# Recon KPI Email Notifier

A small n8n workflow that turns a field survey submission into an instant, personalized HTML
receipt email — so the person who just submitted a cash-reconciliation report gets an immediate,
readable confirmation of exactly what was recorded, instead of waiting on someone to check a
spreadsheet.

## How it works

```
KoBoToolbox submission ──▶ look up submitter in tracker sheet (by username)
                                       │
                                       ▼
                          build HTML KPI summary email
                          (district, branch breakdown, cash totals — every
                           field read through a null-safe helper so a
                           question the submitter skipped can't crash the run)
                                       │
                                       ▼
                              send the email ──▶ delete it from the
                                                  sending account's mailbox
```

1. **KoBoToolbox trigger** fires on every new submission of a cash-sales survey.
2. The submitter's row is looked up in a tracking spreadsheet by their KoBo username — not an
   email address typed anywhere, so there's nothing to spoof or mistype.
3. A code node builds a branded HTML email: district and branch/shop breakdown, individual and
   split totals, each line only rendered if the submitter actually answered that question.
4. The email is sent via Gmail, then immediately deleted from the sending account's own mailbox —
   the recipient keeps their copy; the shared automation inbox doesn't accumulate one sent copy
   per submission.

## Why this exists

A reconciliation survey with dozens of line items is easy to mis-key, and the submitter has no
way to catch that until someone downstream flags a mismatch — often days later. An immediate
"here's exactly what we recorded" email closes that loop while the numbers are still fresh in the
submitter's head, at zero marginal cost per submission.

## Design notes

- **Null-safe by construction.** Every value read from the survey response goes through a helper
  that treats a skipped question as absent rather than throwing — a single incomplete submission
  can't take down the whole automation.
- **Only non-empty lines render.** A branch with no reported sales doesn't show up as a zero row
  cluttering the email; it's omitted entirely.
- **Lookup by username, not email.** The recipient address comes from the tracker sheet via a
  trusted join key (the platform's own submitter identity), not from anything the submission
  itself could claim.

## Stack

n8n · KoBoToolbox · Google Sheets · Gmail

## Files

- `workflow.json` — the full n8n workflow export. After importing (**Workflows → Import from
  File**), point the **Get row(s) in sheet** node at your own tracker spreadsheet (replace
  `YOUR_GOOGLE_SHEET_ID_HERE`) and set your own KoBoToolbox and Gmail credentials — credentials
  are never included in an n8n export.
