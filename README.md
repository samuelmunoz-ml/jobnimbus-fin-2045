# JobNimbus - First payment link after approval (FIN-2045)

The screen an admin sees once their JobNimbus payments account (Stripe Connect) is
approved. It lists open invoices that went out before Pay now existed and lets the
admin send a payment link on each one. The goal is the first payment, not the
approval.

## Pages

- **`index.html`** - a **scenario picker**. Choose the account status and variations,
  then "Open the prototype" takes you to the product screen with those settings
  applied. Share this link for feedback.
- **`product.html`** - the **product screen**. Reads the chosen variations from URL
  query params (see below), so any scenario is a shareable link on its own. A
  "Prototype options" chip links back to the picker.
- **`concept.html`** - an all-in-one **interactive concept** with live toggles and an
  engineer-notes overlay on the page itself (an alternative to the picker flow).

All three are single self-contained HTML files, no build step (fonts load from Google Fonts).

## Run locally

```bash
python3 -m http.server 8000
```

then visit http://localhost:8000.

## Product URL parameters (`product.html`)

| Param | Values | Default |
| --- | --- | --- |
| `status` | `enabled`, `payouts_pending`, `requirements_due` | `enabled` |
| `scope` | `single`, `account` (account adds a Location column) | `single` |
| `ach` | `1` (card + bank account/ACH), `0` (card only) | `1` |
| `ein` | `same`, `different` (drives the add-location card) | `same` |
| `eligible` | `0`, `3`, `5`, `12` (how many open invoices need a link) | all eligible |
| `state` | `paid`, `processing`, `failed` (initial state on the newest row) | none |

Example: `product.html?status=payouts_pending&scope=account&ach=0&eligible=3&state=paid`

In production these come from the backend account, feature flags, and Stripe payment
events; the params just stand in for them so reviewers can explore each case.

## Notes for engineering

- Gating in production: admin only (`access.settings`), behind the `stripe-payments` flag.
- Eligibility is one function: balance > 0, sent before the approval date, finalized
  (not void or draft), no existing payment link, in scope.
- The send flow reuses the existing Send payment request flow, prefilled.
- Every payment carries `job_id` and `location_id` metadata.
- `paid`, `processing` and `failed` row states come from Stripe payment events (webhooks).
- Card fees (surcharging) are owned by FIN-1662 and set per location; this screen links out.
- Analytics: `payments_first_link_sent` fires once per location on the first send;
  later sends fire `payments_link_sent`.

### Open items to confirm

- Scope: one location or the whole account.
- Activation window: 14 days (ticket) vs 45 days (success team cycle), and whether
  ACH counts at initiated or settled.
- New-account processing limit with Stripe.
