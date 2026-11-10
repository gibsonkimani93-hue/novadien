# NOVADIEN — catalogue and order-workflow build

## Run locally

1. Install Node.js 18 or newer.
2. Open PowerShell in this project folder.
3. Run `npm install`.
4. Copy `.env.example` to `.env` and set `ADMIN_PASSWORD` and `SESSION_SECRET` (instructions below).
5. Run `npm start`.
6. Open `http://localhost:3000`.

## Administrator security configuration

The administrator API is now protected by a signed, expiring bearer token. The CEO workspace asks for the administrator password when opened. Tokens expire after 12 hours and are held in the browser session storage. Five failed login attempts from one IP trigger a 15-minute temporary lockout. Public order submission and tracking endpoints also have basic in-memory rate limits. Security response headers are enabled; HSTS is enabled when `NODE_ENV=production`.

Create a strong unique password and signing secret. In PowerShell, from the project folder:

```powershell
Copy-Item .env.example .env
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Put the generated value into `SESSION_SECRET` in `.env`, and set a unique `ADMIN_PASSWORD` (at least 16 characters). Never upload `.env` to GitHub. On Render, add both values under the service's Environment settings instead of uploading a `.env` file. Also set `NODE_ENV` to `production`.

This is an initial security improvement, not a full enterprise identity system. Keep the admin password private. Rotate it if it is exposed. A signed token is not a substitute for HTTPS; only use the production service over HTTPS.

## Current workflow

1. Customer views the four product catalogue entries and submits a request with delivery details and an exact map pin.
2. A verified supplier covering the delivery area can receive an in-app order notification.
3. The supplier can accept or reject an assigned request using the current supplier portal.
4. A supplier can share actual browser GPS coordinates during an active delivery.
5. Customer tracking looks up the order using the reference and the phone number used at order time.

## Work completed in this update

- Added administrator password login and signed 12-hour admin API tokens.
- Protected all `/api/admin/*` routes except the login endpoint.
- Added basic failed-login throttling and rate limits for public order submission and tracking.
- Added security response headers and removed the Express technology header.
- Added `.env.example` and a PostgreSQL schema starter in `DATABASE-PLAN.sql`.
- Added an administrator logout control.
- Kept the four catalogue product images and request-order controls.
- Removed delivery-fee and supplier-payout fields and controls; delivery records now track assignment and completion only.

## Still required before public launch with real customer data

**Do not deploy this build as a public live ordering service yet.** The following items are not completed:

1. **Permanent database:** the application still reads/writes `data.json`. `DATABASE-PLAN.sql` is a planning starter only; PostgreSQL is not connected. Render's temporary filesystem can lose JSON data on redeploy/restart. Do not rely on it for orders.
2. **Supplier authentication:** the multi-supplier directory has been removed. One supplier is configured with `SUPPLIER_NAME`, `SUPPLIER_PHONE` and a private `SUPPLIER_ACCESS_KEY` environment variable. Protect that key carefully, rotate it if exposed, and review the supplier portal authentication design before public production use.
3. **Customer privacy and access:** customer order lookup still uses order reference plus phone number, and GPS/location handling needs a full privacy and access-control review. Add retention/deletion rules, backups, audit review and incident procedures.
4. **Operational notifications:** supplier notifications are in-app only. SMS, WhatsApp, email or push alerts are not configured.
5. **Production testing:** run end-to-end tests, concurrency tests, backup/restore tests and security testing. The basic in-memory rate limits reset when the server restarts and are not a substitute for a managed rate-limiting service.
6. **Medicine compliance:** verify business and supplier licensing, product identity, labels, permitted claims, stock, prescription requirements and all applicable Kenyan medicines, consumer-protection and data-protection requirements before accepting or fulfilling orders. Product catalogue information must be checked against approved packaging and qualified professional guidance.
7. **Payments and maps:** no payment gateway/webhooks are connected. Map tiles, geocoding and routing currently depend on public OpenStreetMap/Nominatim/OSRM services; review their terms and operational suitability before production use.

## Deployment status

The project is a Node.js/Express application. Render can use build command `npm install` and start command `npm start`, with its assigned `PORT`. However, use only a private/staging deployment with test data until the outstanding items above are complete. Set secrets through the hosting provider's environment settings; do not commit secrets.


## Single-supplier configuration
Set `SUPPLIER_NAME`, `SUPPLIER_PHONE`, and a long random `SUPPLIER_ACCESS_KEY` in the hosting provider's environment settings. The supplier portal accepts only that private access key; do not share it with customers or place it in public source code. All new orders are routed to the one configured supplier, and the supplier can accept/reject orders and share GPS for active deliveries. Multi-supplier registration and assignment are disabled.

This build still stores records in local `data.json`. Do not use it for real customer orders on an ephemeral hosting filesystem until PostgreSQL persistence, backups, privacy review, secure notification delivery, and end-to-end tests are completed.


## Customer-facing Novadien Assistant

A floating “Ask Novadien” chat assistant is included on the customer storefront. It can answer general catalogue and order-process questions using the current active product catalogue. It does not read private order records, does not store chat transcripts, and directs customers to the existing order-tracking flow for order-specific information. It is deliberately restricted from diagnosing, prescribing, recommending treatments, or giving dosage/medical advice.

To enable AI replies, add `OPENAI_API_KEY` to your local `.env` file or hosting provider's private Environment settings. The key is used only by the Express server and is never exposed to the browser. `OPENAI_MODEL` defaults to `gpt-5.5` and can be changed to a model enabled for your API account. Without the key, the chat panel remains visible but explains that AI support is not connected. API usage may incur charges; set usage limits in your API account and never commit `.env` to GitHub.

The assistant endpoint is rate-limited and accepts a maximum 1,200-character message. Do not ask customers to submit medical records, passwords, payment-card details, or identity numbers. The assistant is customer-support software, not a clinical service.


### Enable the customer AI assistant

The storefront chat is connected to the OpenAI Responses API through the Express server. Set `OPENAI_API_KEY` in the local `.env` file or in your hosting provider's private environment settings. Keep the key out of `public/`, browser JavaScript, screenshots, GitHub commits, and customer messages. Set `OPENAI_MODEL` to a model enabled for your API project (the included default is `gpt-5.5`). Restart/redeploy the server after changing environment variables. Check `GET /api/customer-assistant/status` to confirm whether a key is configured; this endpoint does not reveal the key. The assistant keeps only a short conversation history in the customer's current browser page and does not save chat transcripts. API usage is billed separately from a ChatGPT subscription; configure a budget/usage limit in the API account.


## SEO readiness added
- Search-friendly homepage title, description, author and social preview metadata.
- Dynamic `robots.txt` and `sitemap.xml` using `PUBLIC_SITE_URL` (or the incoming host as a fallback).
- Dynamic Organization and WebSite JSON-LD on the homepage.
- `/health` endpoint for deployment checks.
- See `SEO-DEPLOYMENT.md` for Render and Google Search Console steps.

Set `PUBLIC_SITE_URL` to the final HTTPS domain in Render before submitting the sitemap. A branded Google result is not guaranteed; Google determines crawling, indexing and ranking.
