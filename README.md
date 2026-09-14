# UNILAG MBA SA — Annual Dinner & Gala Ticketing

Ticketing site and backend for the UNILAG MBA Students' Association Annual Dinner & Gala.

## Structure

```
frontend/   Guest-facing landing page (static HTML/CSS/JS)
backend/    Paystack webhook handler + Google Sheets sync (to be built)
```

## Frontend

`frontend/index.html` is a static page — no build step. It collects:

- Full name, email, phone
- **Year** (Year 1 — ₦35,000 / Year 2 — ₦40,000)
- **Programme** (Full-Time / Part-Time)
- **Stream** (1 / 2 / 3 — Part-Time only, shown conditionally, grouping only, does not affect price)

Currently the form only shows a local confirmation message (see the `TODO` in the `<script>` block at the bottom of `index.html`). It still needs to be wired to a real endpoint.

## Backend (to be built)

Intended flow:

1. Frontend form submit → POST to a backend endpoint (this repo, hosted on Render) with `{ name, email, phone, year, programme, stream }`.
2. Backend calls Paystack's Initialize Transaction API, passing `year`, `programme`, and `stream` as `metadata`, and redirects the guest to Paystack's checkout.
3. On successful payment, Paystack calls the backend's webhook (`charge.success` event).
4. Webhook handler verifies the event, pulls `data.metadata`, and writes a row to the master Google Sheet: name, email, phone, tier/year, programme, stream, payment reference, amount, status.
5. Each course rep's Sheet (formula-linked to the master, filtered by year/programme/stream) reflects the new row automatically.

### Environment variables you'll need on Render

```
PAYSTACK_SECRET_KEY=
GOOGLE_SERVICE_ACCOUNT_EMAIL=
GOOGLE_SERVICE_ACCOUNT_PRIVATE_KEY=
MASTER_SHEET_ID=
```

### Notes

- Verify the Paystack webhook signature (`x-paystack-signature` header) before trusting any payload.
- For Full-Time guests, `stream` will be empty/null — handle that explicitly rather than writing a blank string.
- Keep ticket-tier pricing (₦35,000 / ₦40,000) authoritative on the backend, not the frontend, so a guest can't tamper with the amount before payment.
