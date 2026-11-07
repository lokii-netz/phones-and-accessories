A Canadian phone and accessories marketplace built with React, Vite, Express, and PostgreSQL. Product prices and checkout are in Canadian dollars. The server calculates inventory, shipping, and provincial tax; customer and seller actions are stored in PostgreSQL.

## Run locally

Requires Node.js 22.13 or newer and a PostgreSQL database. For a free demo, create a Neon Free project and copy its pooled connection string. In PowerShell, from the project directory:

```powershell
Copy-Item .env.example .env
npm install
npm run dev
```

Set `DATABASE_URL` in `.env` to the Neon connection string, then open <http://localhost:3000>. The server creates the required tables and sample catalog on first startup.

To make the first admin account, set `ADMIN_EMAIL` and an `ADMIN_PASSWORD` of at least 14 characters in `.env`, then restart the server. Sign in with that email and password; the server creates the configured administrator account automatically. Keep both values private.

Useful commands:

```powershell
npm run lint
npm run build
npm start
```

`npm start` serves the production build. Run `npm run build` first. The server requires `DATABASE_URL`.

## Included marketplace features

- Searchable catalog with 50 sample listings in each of the 18 phone and accessory categories (900 new seed listings), colour variants, category filters, and CAD pricing. These are illustrative catalogue entries, not confirmed supplier inventory; verify model availability, colour, specifications, and price before listing products for sale.
- Product detail views, favourites, a cart, quantity controls, inventory checks, and customer order history.
- Customer registration and sign-in using scrypt password hashes and secure, HTTP-only server sessions.
- Canadian address fields and postal-code checks; province-specific estimated sales tax and standard/express shipping.
- Standard and express delivery estimates are offered to Canadian provinces and territories; they are illustrative flat-rate estimates, not carrier quotes or booked/tracked shipments.
- Seller applications, administrator approval, seller product submissions, and product moderation before publication.
- Seller inventory and order views, plus an administrator dashboard for sellers, listings, orders, and sales totals.
- Cash-on-delivery order placement. Orders using Stripe or PayPal remain unpaid until the provider confirms payment.

## Configure real payments

Payment secrets belong only in the server's environment variables. Never add them to React code, `VITE_*` variables, or source control. The app deliberately has no fake payment-success path.

**Stripe:** Create a Stripe account and use a test secret key (`sk_test_…`) while testing. Set `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, and `PUBLIC_BASE_URL` to your public HTTPS URL. Register `https://your-domain/api/webhooks/stripe` in Stripe and subscribe to `checkout.session.completed`. The payment is recorded as paid only after the signed webhook confirms it. Test card numbers and test-mode behavior are provided by Stripe; live charges require Stripe account approval and live credentials.

**PayPal:** Create a PayPal developer sandbox app. Set `PAYPAL_CLIENT_ID` and `PAYPAL_CLIENT_SECRET`; leave `PAYPAL_MODE=sandbox` for testing. Switch to `PAYPAL_MODE=live` only with PayPal live credentials, an HTTPS deployment, and verified merchant setup. PayPal payment capture is performed server-side and the amount and CAD currency are checked before an order is marked paid.

**Cash on delivery** works without a payment-provider account. It confirms an order with the status `confirmed`; it does not represent an online card payment.

Use the blank values in `.env.example` as a checklist. Provider credentials and a publicly reachable HTTPS deployment are external requirements; this local application does not include merchant accounts, email delivery, or payment-provider credentials.

## Seller and administrator setup

Customers can apply to become a seller from the account menu. Admins review applications and each new/edited seller listing before it becomes visible. Seller listings currently accept HTTPS image URLs; image upload/storage requires an external storage service and is not included. Sellers can create and remove listings; new or edited listings are held for review.

The admin account is configured using `ADMIN_EMAIL` and `ADMIN_PASSWORD` as described above. Use a unique random password and store it in the deployment platform's secret manager. Admin controls cover seller/listing approval and order status. Refunds, email notifications, full customer support workflows, automated password-reset delivery, and comprehensive promotion/category management are not connected yet.

## Deploy

### Netlify (free demo)

The repository root contains `netlify.toml` and a Netlify Function for the Express API. Connect this repository to Netlify and deploy the `main` branch. In the Netlify site settings, add `DATABASE_URL` using the pooled connection string from a Neon project, plus `ADMIN_EMAIL` and a unique `ADMIN_PASSWORD` of at least 14 characters to initialize the admin account. `PUBLIC_BASE_URL` is optional unless enabling online payments; set it to the final HTTPS site URL before configuring Stripe or PayPal. Keep database and payment credentials in Netlify's environment-variable settings, not in source control.

The static storefront is served by Netlify; `/api/*` requests are forwarded to the function. Neon stores accounts, orders, inventory, and listings outside the function filesystem. Seed products are inserted once per database and are illustrative, not verified inventory. The free plans have provider quotas and no production availability guarantees; verify current limits and arrange backups before relying on stored data.

The Render blueprint is also available at the repository root in `render.yaml`. For another Node host, use Node.js 22.13+, run `npm install` and `npm run build`, set `DATABASE_URL`, and start with `npm start`.

## Launch-readiness notes

This is a functional marketplace foundation, not a fully audited production commerce platform. The included catalog is illustrative sample data; ratings are not shown for sample products. Replace sample listings with authorized, accurate seller inventory. Configure and test payment webhooks, merchant accounts, tax rules, actual delivery providers, email, backups, privacy/consumer notices, refund processes, and abuse/support operations before going live. Canadian tax treatment can depend on registration and product/shipping details; have a Canadian tax professional verify rates and rules for your business.
