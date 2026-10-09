# LITTO MOTTOS — Vehicle Marketplace

A responsive automotive marketplace starter with a premium showroom UI, SQLite-backed REST API, customer accounts, admin catalogue management, orders, service bookings, interest events and PWA caching.

## Requirements
- Node.js 20 or newer
- npm
- A modern browser

## Run locally
1. Extract the ZIP and open a terminal in the `litto-mottos` folder.
2. Install dependencies: `npm install`
3. Copy `.env.example` to `.env` and set a long, random `JWT_SECRET` (at least 32 characters). Set a strong `ADMIN_PASSWORD` of at least 12 characters.
4. Create the initial administrator: `npm run init-admin`
5. Start the app: `npm start`
6. Open `http://localhost:3000`. Admin console: `http://localhost:3000/admin`.

SQLite database is created automatically at `data/litto-mottos.sqlite` on first start. Sample vehicles are inserted when the vehicle table is empty. Back up the `data` folder regularly.

## Included functionality
- Responsive landing page, searchable vehicle catalogue, category filters, vehicle detail/specification dialog and local cart.
- Registration and email/password login with bcrypt password hashing and JWT sessions.
- Admin-only vehicle creation with optional image upload and acquisition cost, availability update and delete; dashboard counts, sales reporting, orders, bookings, customers, administrator accounts and customer-interest events.
- Vehicle order-request creation and status updates. Booking requests for wedding, self-drive/tours and imports.
- Checkout captures a buyer contact number, creates an initial order notification and opens prepared WhatsApp links for the three sales contacts. Buyers and admins can continue the order conversation in their dashboards.
- PWA manifest and service worker cache for the app shell; browser online/offline indicator and a local retry queue for booking requests.
- SQLite schema with foreign keys, validation checks, timestamps and indexes.

## API overview
- `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me`
- `GET /api/vehicles`, `GET /api/vehicles/:id`
- `POST /api/interests`
- `POST /api/bookings`, `GET /api/bookings`
- `POST /api/orders`, `GET /api/orders`
- Order conversations: `GET /api/orders/:id/messages`, `POST /api/orders/:id/messages`
- Admin: `GET /api/admin/stats`, `GET /api/admin/users`, `GET /api/admin/interests`
- Admin vehicle CRUD: `POST /api/admin/vehicles`, `PUT /api/admin/vehicles/:id`, `DELETE /api/admin/vehicles/:id`
- Admin team: `GET /api/admin/team/admins`, `POST /api/admin/team/admins` (initial password minimum 8 characters; 12+ recommended)
- Admin profile: `GET /api/admin/profile`, `PATCH /api/admin/profile` (password updates require the current password and a 12-character minimum)
- Admin reports and operations: `GET /api/admin/reports/sales`, `PATCH /api/admin/orders/:id/payment`, `PATCH /api/admin/orders/:id`, `PATCH /api/admin/bookings/:id`
- Admin operating expenses: `GET /api/admin/expenses?months=12`, `POST /api/admin/expenses` (charts summarize saved amounts by month and category)
- Admin image uploads: `POST /api/admin/uploads/vehicle-image` with multipart fields `image` and normalized `plateRegion`; existing listings use `POST /api/admin/vehicles/:id/brand-plate`. Both composite the company logo into the admin-marked plate area.
- `GET /api/health`

Authenticated endpoints require `Authorization: Bearer <token>`. Admin routes require a JWT with role `admin`. Sales revenue counts paid orders; profit includes only paid orders with recorded vehicle acquisition costs. Payment status changes are operational records, not charges or refunds.
WhatsApp order messages are prefilled links and are sent only after the buyer opens WhatsApp and presses send. Automated WhatsApp delivery requires a configured WhatsApp Business API integration.

## Important production work before launch
- **Google sign-in:** the UI explains that Google OAuth must be configured. Add OAuth credentials, server-side state/nonce validation and a verified Google identity flow before enabling it; never trust a client-provided email as proof of identity.
- **Payments:** checkout currently creates an order request only. Admin payment status changes are manual records; no card is collected or payment/refund processed. Integrate a reputable payment provider's hosted checkout/tokenized SDK and verify payment webhooks server-side before marking an order paid. Never store card numbers or CVV.
- **Offline sync:** app shell caching and a retry queue for service booking requests are included. Offline orders are not automatically submitted, and the booking retry queue is stored locally. Review consent, duplicate prevention and conflict handling before production use.
- **Images/media:** demo vehicles use remote Unsplash images. An admin marks the plate region on existing photos and new uploads; the system then saves a branded image under `public/uploads`. Use licensed images and move uploads to durable object storage before multi-instance production deployment.
- **Security/operations:** use HTTPS, a strong secret manager, strict production CORS allow-list, CSRF protections where applicable, account lockout/monitoring, audit logs, database backups, privacy notice/consent, retention rules, server-side pagination, and tested migrations. Configure Helmet CSP rather than leaving it disabled for the inline-style starter. Use a managed database and durable object storage for multi-instance cloud deployments.
- **Commercial workflow:** validate availability, pricing, tax, delivery, deposits and booking rules with the business before accepting real orders. Seed technical specifications are illustrative; confirm exact trim/year specifications before publication.

## Project structure
```
public/
  index.html          customer showroom
  admin.html          administrator console
  admin-profile.html administrator profile and password settings
  css/styles.css      responsive visual system
  js/app.js           customer interactions and API calls
  js/admin.js         administrator interactions
  js/admin-tools.js   sales, payment and administrator workflows
  js/admin-profile.js administrator profile interactions
  css/admin-tools.css admin dashboard additions
  css/admin-profile.css administrator profile styling
  css/motion.css     reduced-motion-aware showroom animation
  images/logo.svg     original simple brand mark
  manifest.webmanifest
  sw.js
src/
  server.js           Express REST API
  create-admin.js     secure initial admin setup
  db/database.js      SQLite setup and sample inventory
  db/schema.sql       tables, constraints and indexes
  middleware/auth.js  JWT and role authorization
.env.example
package.json
README.md
```

The supplied reference image is included as `public/images/reference-inspiration.png` for project context; the interface is independently styled. This project is a working starter, not a payment-certified or production-audited commerce platform.
