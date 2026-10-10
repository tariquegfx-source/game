# RealGame — PostgreSQL-backed play-money website

This version replaces the local JSON file with PostgreSQL, adds Docker/Render deployment templates, and provides a server-side Sportmonks Cricket API adapter. All balances are non-redeemable play tokens. Real-money wagering, deposits and cash-outs remain disabled.

## What is implemented

- PostgreSQL schema and transaction-backed balance updates for demo credits and demo rounds.
- Password hashes using Node.js `scrypt`, HTTP-only SameSite session cookies, role-protected admin page/APIs, audit events, account suspension.
- Server-side Sportmonks endpoints: `GET /api/sports/cricket/livescores` and `GET /api/sports/cricket/fixtures/:id`.
- Homepage attempts to load real cricket scores from the backend and polls every 30 seconds. If the API token is absent or the provider call fails, the page keeps its demo fixture view and shows a status note.
- Health endpoint checks PostgreSQL: `GET /api/health`.
- Dockerfile, Docker Compose for local testing, and Render web-service blueprint.
- Payment/cash-out routes explicitly return `501`; they do not collect or move real money.

## Local run with Docker

1. Install Docker Desktop / Docker Engine.
2. Copy `.env.example` to `.env` and set strong values for `POSTGRES_PASSWORD`, `ADMIN_USERNAME`, `ADMIN_PASSWORD`. Optionally add `SPORTMONKS_API_TOKEN` from your Sportmonks account.
3. Run `docker compose up --build`.
4. Visit `http://localhost:3000`. Test health at `http://localhost:3000/api/health`.
5. To test live scores, configure `SPORTMONKS_API_TOKEN`, restart the app, and inspect `/api/sports/cricket/livescores`. A successful API response confirms the token/feed works; no credentials are bundled in this project.

## Deploy on Render (public web service)

1. Push this folder to a private GitHub repository.
2. In Render, create a managed PostgreSQL database in the same region as the web service.
3. Create a new Blueprint or Web Service from the repository. The included `render.yaml` creates the web service; set `DATABASE_URL` to the internal connection string from your managed PostgreSQL instance.
4. Set `NODE_ENV=production`, a unique `ADMIN_USERNAME`, a strong `ADMIN_PASSWORD`, and `SPORTMONKS_API_TOKEN` in the Render environment. Do not commit `.env` or API keys.
5. Deploy and verify `/api/health`, registration/login, wallet ledger, admin access, and `/api/sports/cricket/livescores`.
6. Add your custom domain in Render's dashboard, configure the DNS records Render provides at your domain registrar, and wait for TLS certificate provisioning.

I cannot publish this to a public URL or configure your domain from this workspace because I do not have access to your hosting, database, GitHub, domain registrar, or provider accounts. Do not share passwords/API secrets in chat; add them directly to your hosting environment settings.

## Database details

Tables are created on application startup: `users`, `sessions`, `bets`, `wallet_ledger`, `audit_events`. `db/schema.sql` contains the schema for review/reference. Use managed PostgreSQL backups and point-in-time recovery before production. Run an independent security review before exposing accounts publicly; add rate limiting, CSRF protection, email verification, admin MFA, monitoring, and session revocation/rotation.

## Live cricket provider

Sportmonks API token is read only by the server from `SPORTMONKS_API_TOKEN`. It is never included in the browser source. The score feed is polled every 30 seconds by the page; respect the rate limits and licensing terms on your subscription. The endpoint returns scores, not official betting odds. Any odds currently present in the demo match table are illustrative only.

## Payments, wagering and cash-outs

These routes intentionally remain disabled. Razorpay's published terms prohibit using its services for online gambling/sports bets. Do not collect deposits or offer withdrawals for wagering until the business has obtained applicable legal authorization and a payment provider has explicitly approved the exact use case. For a lawful non-wagering product, implement the provider's hosted checkout and signed webhook flow only after approval; never mark an order paid based on the browser redirect alone.

## Important limitations

- Sportmonks credentials are not supplied and the live feed cannot be verified until you add your token.
- A custom domain, production database and public hosting cannot be provisioned without your own accounts and DNS access.
- This is a deployable starter, not a completed production betting platform or audited financial system.
