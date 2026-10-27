# Research Club Tickets

Next.js 14 app for event registration with waitlists, paid events, themed tickets, and an admin area. Google Sheets holds all data.

## 1. Google Sheets setup

1. Open console.cloud.google.com and create a project.
2. Go to APIs & Services, Library, and enable **Google Sheets API**.
3. Go to IAM & Admin, Service Accounts, Create service account. Name it `club-tickets`. Skip the optional role steps.
4. Open the account, go to Keys, Add key, Create new key, JSON. A file downloads.
5. From that file, copy `client_email` into `GOOGLE_SERVICE_ACCOUNT_EMAIL` and `private_key` into `GOOGLE_PRIVATE_KEY`.
6. Create a blank Google Sheet. Copy the ID from its URL (`/spreadsheets/d/<ID>/edit`) into `SHEET_ID`.
7. Click Share on the sheet and add the `client_email` address as **Editor**.

The app creates the `Events`, `Registrations`, and `Updates` tabs with headers on first load. Registration statuses are `requested`, `pending_payment`, `confirmed`, `waitlist`, and `cancelled`. You can edit a status cell by hand. Opening the admin page re-runs the seat logic, so setting a row to `cancelled` promotes the next waitlisted person.

## 2. Payments (SSLCommerz)

Stripe does not onboard Bangladesh businesses, so the app uses SSLCommerz hosted checkout.

1. Create a sandbox account at developer.sslcommerz.com. The sandbox store ID and password arrive by email.
2. Put them in `SSLCZ_STORE_ID` and `SSLCZ_STORE_PASSWORD`. Keep `SSLCZ_LIVE=false` while testing.
3. For live payments, apply for an SSLCommerz merchant account, use the live store credentials, and set `SSLCZ_LIVE=true`.
4. In the merchant panel, set the IPN URL to `https://YOUR-SITE/api/pay/ipn`. The app also sends it with each payment.

Flow: a paid registration holds a seat as `pending_payment` for 60 minutes. The attendee pays on SSLCommerz. The return page and the IPN webhook both verify the payment with SSLCommerz, check the amount against the event price, and mark the row `confirmed`. The ticket then appears on the ticket page.

## 3. Environment variables

See `.env.example`. `SITE_URL` must be the public URL with no trailing slash. `SESSION_SECRET` is any long random string. `ADMIN_PASSWORD` is the single admin login.

## 4. Run locally

```
cp .env.example .env.local
npm install
npm run dev
```

## 5. Deploy on Vercel

1. Push this folder to a GitHub repo.
2. In Vercel, choose Add New, Project, and import the repo.
3. Under Environment Variables, add every key from `.env.example`. Paste `GOOGLE_PRIVATE_KEY` exactly as it appears in the JSON file, with the `\n` sequences.
4. Deploy. Copy the production URL into `SITE_URL`, then redeploy.
5. Open `/admin`, sign in, and create your first event.

## How it works

- `lib/sheets.ts` has all sheet reads and writes plus `rebalance()`, the capacity and waitlist logic. Seats go to `requested` and `waitlist` rows in sheet row order. That makes results the same when two people register at once.
- A cancellation, a capacity increase, or an expired payment hold triggers `rebalance()`. Free events promote straight to `confirmed`. Paid events promote to `pending_payment`.
- Tickets are PNG images drawn from the event theme. The ticket ID seeds the shape pattern, so every ticket looks different within the event palette. Route: `/api/ticket/<id>/image`.
- Animation lives in `components/Pop.tsx`. It uses spring scale entrances and turns off when the visitor prefers reduced motion.

## Limits

- No emails. Attendees return to their ticket page for waitlist promotions and updates. Share the ticket link when you can.
- Ticket images use the built-in Latin font, so names in other scripts will not render.
- Google Sheets allows about 60 reads per minute per service account. That covers a club-sized audience. Heavy launch traffic needs a database.
