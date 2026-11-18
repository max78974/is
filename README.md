# Ice Scoop’s

Ice Scoop’s is a mobile-first ordering website for a single outlet. The storefront and staff dashboard use HTML, CSS, and vanilla JavaScript. An Express server stores orders, menu edits, audit history, and staff sessions in SQLite. The original supplied logo is in `public/assets/logo.png`. The starter catalogue comes with 15 flavour photos.

## Run locally

Install Node.js 24 or newer. In PowerShell, run these commands from the project folder.

```powershell
npm ci
Copy-Item .env.example .env
npm start
```

Open [Storefront](http://localhost:3000/) or [Staff dashboard](http://localhost:3000/admin.html). The database and starter catalogue are created on first launch. Pay at Counter works immediately. Pay Online becomes available after Razorpay is configured.

## Staff accounts

Set `ADMIN_EMAIL` and a unique `ADMIN_PASSWORD` of at least 12 characters in `.env` before the first start to create the owner. Remove the bootstrap credentials after the server creates the account. Existing accounts stay in SQLite.

Use the CLI to create another owner or staff account. Supply its password as `ICE_SCOOPS_ADMIN_PASSWORD` in the current shell and remove the environment variable after use.

```powershell
npm run admin:create -- staff staff@example.com
```

Owners can manage the catalogue and store settings. Staff can manage orders, record counter payments, and operate the kitchen view. Orders are grouped by the outlet’s India business date. Sales totals count collected payments on eligible orders.

## Configure Razorpay

1. Add `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, and `RAZORPAY_WEBHOOK_SECRET` to the server’s environment.
2. In Razorpay, enable automatic capture and configure the webhook for `https://your-store.example/api/razorpay/webhook`. Subscribe to `payment.captured`, `payment.failed`, and `refund.processed`.
3. Match the webhook secret to the server value. Restart the server and verify a Test Mode success, failure, and dismissed checkout. Confirm one paid order and one kitchen ticket for a captured payment.
4. Use the live key pair and matching live webhook only after the test flow succeeds. Payment methods in Checkout are the methods enabled for your Razorpay account.

The server recalculates each total in paise and verifies the gateway order, amount, signature, and captured payment before it records an online order as paid. [Standard Checkout](https://razorpay.com/docs/payments/payment-gateway/web-integration/standard/integration-steps/) · [Webhook validation](https://razorpay.com/docs/webhooks/validate-test/).

## Kitchen tickets and receipts

Counter orders appear as payment pending. By default, staff must record payment before printing the kitchen ticket. Owners can release tickets at order time for counter orders in the staff settings. Online tickets become available after capture.

Select 58 mm or 80 mm when opening a ticket. In the browser print dialogue, select the matching thermal paper size, print at 100%, and turn off browser headers and footers. If the printer has narrow physical margins, use its default printable area. Run a sample ticket before using the printer for service.

Customers can reopen the latest receipt on their device, print it, or download a self-contained HTML copy. The current payment status appears when the receipt refreshes.

Use `/api/qr` on the configured store origin to download a PNG for a kiosk sign. Scan the printed QR before putting the sign out.

## Production deployment

1. Deploy this directory to a Node host with persistent local storage. Run `npm ci --omit=dev` and start `npm start` under a service manager as an unprivileged user.
2. Terminate HTTPS on a trusted reverse proxy. Set `NODE_ENV=production`, `APP_ORIGIN=https://your-store.example`, `HOST=127.0.0.1` for a proxy on the same host, and `TRUST_PROXY_HOPS=1` for exactly one proxy. Configure the proxy to overwrite forwarded headers and pass through streaming responses on the staff events endpoint. [Express proxy configuration](https://expressjs.com/en/guide/behind-proxies/).
3. Point `DATABASE_PATH` to a writable, persistent location outside `public/`. Set a stable 32-character-or-longer `ORDER_ACCESS_SECRET`. Keep the database, secrets and backups accessible only to the service and authorized operators.
4. Use a single application instance for SQLite and live staff notifications. Back up the database regularly using SQLite’s backup interface or a consistent volume snapshot so committed WAL changes are included. [Node SQLite backup](https://nodejs.org/download/release/latest-v24.x/docs/api/sqlite.html#sqlitebackupsourcedb-path-options).
5. Verify the site, staff login, one counter order, receipt retrieval, and KOT printing on the public HTTPS origin. If you enabled Razorpay, verify that the gateway can reach the webhook.

## Source and checks

The storefront is in `public/storefront/`, staff UI in `public/admin/`, API in `server/`, schema in `server/schema.sql`, and seed data in `server/catalog.js`. Run `npm test` for backend checks. Use the responsive tools in a browser to verify the ordering experience at kiosk, phone, and desktop sizes.
