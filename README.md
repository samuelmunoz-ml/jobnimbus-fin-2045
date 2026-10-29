# JobNimbus - First payment link after approval (FIN-2045)

The screen an admin sees once their JobNimbus payments account (Stripe Connect) is
approved. It lists open invoices that went out before Pay now existed and lets the
admin send a payment link on each one. The goal is the first payment, not the
approval.

## Pages

- **`index.html`** - the interactive **review concept**. It adds a "Concept controls"
  bar (not part of the product) so reviewers can switch account status, eligibility,
  scope, ACH and EIN, simulate payment states, and toggle an engineer-notes overlay.
  Use this one for feedback.
- **`product.html`** - the **product screen only**, with the concept scaffolding
  removed and account state set by config constants (see below).

Both are single self-contained HTML files, no build step (fonts load from Google Fonts).

## Run locally

```bash
python3 -m http.server 8000
```

then visit http://localhost:8000 (concept) or http://localhost:8000/product.html.

## Product configuration (`product.html`)

Account state is set with constants at the top of the `<script>`, standing in for
backend account values and feature flags:

| Constant | Values |
| --- | --- |
| `ACCOUNT_STATUS` | `enabled`, `payouts_pending`, `requirements_due` |
| `SCOPE` | `single`, `account` (account adds a Location column) |
| `ACH_ENABLED` | card + bank account (ACH), or card only |
| `SOUTH_BRANCH_EIN` | `same`, `different` (drives the add-location card) |
| `NEW_ACCOUNT_CARD_LIMIT` | `null` until confirmed with Stripe |

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
