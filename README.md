# Welcome to React Router!

A modern, production-ready template for building full-stack React applications using React Router.

[![Open in StackBlitz](https://developer.stackblitz.com/img/open_in_stackblitz.svg)](https://stackblitz.com/github/remix-run/react-router-templates/tree/main/default)

## Features

- 🚀 Server-side rendering
- ⚡️ Hot Module Replacement (HMR)
- 📦 Asset bundling and optimization
- 🔄 Data loading and mutations
- 🔒 TypeScript by default
- 🎉 TailwindCSS for styling
- 📖 [React Router docs](https://reactrouter.com/)

## Getting Started

### Installation

Install the dependencies:

```bash
npm install
```

### Development

Start the development server with HMR:

```bash
npm run dev
```

Your application will be available at `http://localhost:5173`.

## Building for Production

Create a production build:

```bash
npm run build
```

## Authentication

Authentication is powered by [Better Auth](https://www.better-auth.com/). The server config lives in `app/lib/auth.server.ts` and the client in `app/lib/auth-client.ts`.

### Included out of the box

- Email & password sign-up/login
- Email verification and password reset
- Two-factor authentication (TOTP + backup codes)
- Passkeys (WebAuthn)
- Multi-session support
- Organizations with roles (owner, admin, member)
- Teams within organizations
- Member invitations and invite links
- Session management (view/revoke)
- Account deletion
- Password breach checking (Have I Been Pwned)
- Rate limiting on auth endpoints
- Audit logging via database hooks
- Notification system (in-app + email)

### Opt-in (set env vars to activate)

- **Google OAuth** — set `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`
- **SSO (OIDC/SAML)** — configured per-organization in workspace settings
- **Stripe billing** — set `STRIPE_SECRET_KEY` and related vars

### Adding plugins

Better Auth is plugin-based — add to `app/lib/auth.server.ts` (server) and `app/lib/auth-client.ts` (client), then run `npx @better-auth/cli generate` if the plugin adds new tables. See the [Better Auth plugins docs](https://www.better-auth.com/docs/plugins) for available plugins.

## Billing

Billing is powered by [Stripe](https://stripe.com/) via the [`@better-auth/stripe`](https://www.better-auth.com/docs/plugins/stripe) plugin. It's entirely opt-in — billing features are disabled when `STRIPE_SECRET_KEY` is not set.

Set `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, and `STRIPE_PRO_PRICE_ID` to activate.

### What's included

- **Stripe Checkout** — upgrade to Pro via hosted checkout with promotion code support
- **Subscription management** — cancel, resubscribe, and view subscription status
- **Plan details** — displays current plan, price, billing interval, and next billing date
- **Invoice history** — list past invoices with links to hosted receipts and PDF downloads
- **Billing email** — manage the email address Stripe sends receipts to
- **Subscription sync** — sync local subscription state with Stripe on demand
- **Role-gated** — only workspace owners and admins can manage billing
- **Per-organization** — subscriptions are tied to workspaces, not individual users

The billing UI lives in `app/components/workspace-settings-dialog/billing-tab.tsx`. Billing API routes are in `app/routes/api.billing.ts`.

### Webhooks

Stripe webhooks are handled automatically through the Better Auth catch-all route at `/api/auth/stripe/webhook`. The plugin processes `checkout.session.completed`, `customer.subscription.created`, `customer.subscription.updated`, and `customer.subscription.deleted` events.

**Local development:** Use the [Stripe CLI](https://stripe.com/docs/stripe-cli) to forward webhook events to your dev server:

```bash
stripe listen --forward-to http://localhost:5173/api/auth/stripe/webhook
```

The CLI will output a webhook signing secret (`whsec_...`) — set that as `STRIPE_WEBHOOK_SECRET` in your `.env`.

**Production:** Create a webhook endpoint in the [Stripe Dashboard](https://dashboard.stripe.com/webhooks) pointing to `https://yourdomain.com/api/auth/stripe/webhook` and set the signing secret as `STRIPE_WEBHOOK_SECRET`.

### Promotion codes

Promotion codes are enabled on the Stripe Checkout page by default. To create one:

1. Go to **Products > Coupons** in the [Stripe Dashboard](https://dashboard.stripe.com/coupons)
2. Create a coupon (e.g. 20% off for 3 months)
3. Generate a promotion code for the coupon (e.g. `LAUNCH20`)

Customers can enter the code during checkout. Stripe handles validation and discount application.

### Free trials

Free trials are supported per-plan via the `freeTrial` option in the Stripe plugin config in `app/lib/auth.server.ts`:

```ts
plans: [
  {
    name: "pro",
    priceId: process.env.STRIPE_PRO_PRICE_ID!,
    freeTrial: {
      days: 14,
      onTrialStart: async (subscription) => { /* e.g. send welcome email */ },
      onTrialEnd: async ({ subscription }) => { /* e.g. send conversion email */ },
      onTrialExpired: async (subscription) => { /* e.g. downgrade access */ },
    },
  },
],
```

The plugin automatically passes `trial_period_days` to Stripe Checkout, tracks `trialStart`/`trialEnd` in the database, and prevents the same organization from receiving a trial twice.

### Adding plans

A single `pro` plan is configured by default. To add more plans, create a price in your Stripe dashboard and add it to the plugin config in `app/lib/auth.server.ts`.

## Emails

Transactional emails are built with [React Email](https://react.email/) and sent via [Resend](https://resend.com/). Set `RESEND_API_KEY` to activate — without it, emails are logged to the console.

Templates live in `app/emails/` and share a common layout (`app/emails/components/layout.tsx`) for consistent branding and styling.

### Templates

| Template | Trigger |
|---|---|
| `verify-email.tsx` | Sign-up email verification |
| `reset-password.tsx` | Password reset link |
| `verify-new-email.tsx` | Email change verification |
| `otp-code.tsx` | OTP for sign-in, verification, password reset, email change |
| `verify-email-otp.tsx` | Secondary email address verification |
| `welcome.tsx` | New user sign-up |
| `workspace-invitation.tsx` | Invited to a workspace |
| `removed-from-workspace.tsx` | Removed from a workspace |
| `subscription-upgraded.tsx` | Upgraded to a paid plan |
| `subscription-cancelled.tsx` | Subscription cancellation confirmation |
| `notification.tsx` | Generic notification forwarded via email |

### Preview

Run the React Email dev server to preview and edit templates in the browser:

```bash
npm run email:dev
```

### Customization

All templates import shared styles from `app/emails/components/layout.tsx`. Update the layout to change the logo, footer, colors, or font across all emails. Per-template changes are made directly in the template file.

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `BUNNY_DATABASE_URL` | No | Bunny Database / libSQL connection URL. Falls back to `file:dev.db` for local development |
| `BUNNY_DATABASE_AUTH_TOKEN` | No | Auth token for Bunny Database / libSQL remote connections |
| `BETTER_AUTH_URL` | Yes | Base URL of your app (e.g. `https://app.example.com`) |
| `BETTER_AUTH_SECRET` | Yes | Auth secret (min 32 chars). Generate with `openssl rand -base64 32` |
| `GOOGLE_CLIENT_ID` | No | Google OAuth client ID |
| `GOOGLE_CLIENT_SECRET` | No | Google OAuth client secret |
| `RESEND_API_KEY` | No | Resend API key for sending emails. Logs to console if not set |
| `RESEND_WEBHOOK_SECRET` | No | Resend webhook signing secret for delivery tracking |
| `STRIPE_SECRET_KEY` | No | Stripe secret key. Billing features disabled if not set |
| `STRIPE_WEBHOOK_SECRET` | No | Stripe webhook signing secret |
| `STRIPE_PRO_PRICE_ID` | No | Stripe price ID for the Pro plan |
| `BUNNY_API_KEY` | No | BunnyCDN API key for file uploads. Falls back to local filesystem |
| `BUNNY_STORAGE_ZONE` | No | BunnyCDN storage zone name |
| `BUNNY_CDN_URL` | No | BunnyCDN CDN URL |
| `REDIS_URL` | No | Redis connection URL for rate limiting. Falls back to database storage |

## Database

The database layer uses [Drizzle ORM](https://orm.drizzle.team/) with SQLite/libSQL. See the [Drizzle docs](https://orm.drizzle.team/docs/overview) for schema definitions, queries, and migrations.

In local development, a SQLite file (`dev.db`) is used automatically — no setup needed.

In production, create a [Bunny Database](https://bunny.net/database/) and set `BUNNY_DATABASE_URL` and `BUNNY_DATABASE_AUTH_TOKEN`. Bunny Database is libSQL-compatible (like Turso), so Drizzle connects to it natively.

### Commands

| Command | Description |
|---|---|
| `npm run db:push` | Push schema directly to the database (quick, good for local dev) |
| `npm run db:generate` | Generate a versioned migration from schema changes |
| `npm run db:migrate` | Run pending migrations against the database |
| `npm run db:studio` | Open Drizzle Studio to browse your data |
| `npm run db:seed` | Seed the database with sample data |

For local development, `db:push` is the fastest way to iterate. For production, use `db:generate` followed by `db:migrate` so migrations are versioned and reviewable in git.

## Rate Limiting

Rate limiting is enabled by default on all auth endpoints. Stricter limits apply to sensitive routes like sign-in (5 req/min), sign-up (3 req/min), and password reset (3 req/min).

**Storage:** When `REDIS_URL` is set, rate limit counters are stored in Redis via Better Auth's secondary storage. This is recommended for production deployments with multiple instances. Without `REDIS_URL`, counters are stored in the database — fine for single-instance deployments and local development.

## Route Guides

<details>
<summary>Public page (no auth required)</summary>

Pages like login, signup, and marketing pages that anyone can access.

```ts
// app/routes/pricing.tsx
export default function Pricing() {
  return <div>Pricing plans...</div>;
}
```

Register in `app/routes.ts` **before** the `/:slug` workspace route, otherwise it will be caught by the workspace layout:

```ts
route("pricing", "routes/pricing.tsx"),
```
</details>

<details>
<summary>Protected page (authenticated users only)</summary>

Redirects to `/login` if the user has no session.

```ts
// app/routes/dashboard.tsx
import { redirect } from "react-router";
import { auth } from "~/lib/auth.server";
import type { Route } from "./+types/dashboard";

export async function loader({ request }: Route.LoaderArgs) {
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    throw redirect("/login");
  }
  return { user: session.user };
}

export default function Dashboard() {
  // ...
}
```
</details>

<details>
<summary>Workspace page (org members only)</summary>

Child routes of `/:slug` are automatically protected — the workspace layout loader verifies auth and org membership. Access the org via outlet context.

```ts
// app/routes/workspace-reports.tsx
import { useOutletContext } from "react-router";

export default function Reports() {
  const { organization } = useOutletContext<{
    organization: { name: string; slug: string };
  }>();

  return <div>Reports for {organization.name}</div>;
}
```

Register as a child of the workspace layout in `app/routes.ts`:

```ts
route(":slug", "routes/workspace.tsx", [
  // ...existing routes
  route("reports", "routes/workspace-reports.tsx"),
]),
```
</details>

<details>
<summary>Role-restricted page (owner/admin only)</summary>

Use `requireOrgRoleBySlug` in the loader to gate access by role. Returns 403 for unauthorized members.

```ts
// app/routes/workspace-danger-zone.tsx
import { redirect, useOutletContext } from "react-router";
import { auth } from "~/lib/auth.server";
import { requireOrgRoleBySlug } from "~/lib/authorization.server";
import type { Route } from "./+types/workspace-danger-zone";

export async function loader({ request, params }: Route.LoaderArgs) {
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    throw redirect("/login");
  }

  await requireOrgRoleBySlug(session.user.id, params.slug, ["owner", "admin"]);

  return null;
}

export default function DangerZone() {
  const { organization } = useOutletContext<{
    organization: { name: string; slug: string };
  }>();

  return <div>Danger zone for {organization.name}</div>;
}
```

See `app/routes/workspace-example-admin-only.tsx` for a working example. For member-only pages (any role), see `app/routes/workspace-example-members-only.tsx`.
</details>

<details>
<summary>Protected API route</summary>

API routes check the session manually and return JSON errors instead of redirects.

```ts
// app/routes/api.my-endpoint.ts
import { data } from "react-router";
import { getSession } from "~/lib/session.server";
import { requireOrgRole } from "~/lib/authorization.server";
import type { Route } from "./+types/api.my-endpoint";

export async function action({ request }: Route.ActionArgs) {
  const session = await getSession(request);
  if (!session?.user) {
    return data({ error: "Unauthorized" }, { status: 401 });
  }

  const formData = await request.formData();
  const orgId = String(formData.get("orgId"));

  // Optional: check org role
  await requireOrgRole(session.user.id, orgId, ["owner", "admin"]);

  // Handle the request...
  return { success: true };
}
```
</details>

<details>
<summary>Marketing homepage with /home shortcut</summary>

The index route (`/`) shows the marketing page for unauthenticated users and redirects authenticated users to their default workspace.

`/home` provides an explicit URL for the marketing page that authenticated users can visit without being redirected away. Unauthenticated users visiting `/home` are redirected to `/` to avoid duplicate content.

```
/          → logged out: marketing page, logged in: redirect to /:slug
/home      → logged out: redirect to /, logged in: marketing page
```

See `app/routes/home.tsx` and `app/routes/home-page.tsx`.
</details>

## Scheduled Cleanup

A cleanup script removes stale data: expired invite links, expired verification tokens, expired sessions, and unverified accounts older than 7 days.

Run it manually:

```bash
npm run cleanup
```

Or schedule it as a cron job (e.g. every hour):

```
0 * * * * cd /app && npx tsx scripts/cleanup.ts
```

## Deployment

### Prerequisites

Before deploying, set up these services and have your environment variables ready:

1. **Bunny Database** — create a database at [bunny.net/database](https://bunny.net/database/) for `BUNNY_DATABASE_URL` and `BUNNY_DATABASE_AUTH_TOKEN`
2. **Bunny Storage** (optional) — create a storage zone for file uploads (`BUNNY_API_KEY`, `BUNNY_STORAGE_ZONE`, `BUNNY_CDN_URL`)
3. **Push your schema** — run `npm run db:generate && npm run db:migrate` against your production database

### Bunny Magic Containers

A GitHub Actions workflow is included at `.github/workflows/deploy.yml` that builds a Docker image, pushes it to GitHub Container Registry, and deploys to [Bunny Magic Containers](https://bunny.net/magic-containers/).

**Setup:**

1. Create a Magic Containers app in the Bunny dashboard
2. Add these to your GitHub repository settings:
   - **Secret:** `BUNNY_API_KEY` — your Bunny account API key
   - **Variable:** `BUNNY_APP_ID` — your Magic Containers app ID
3. Configure your environment variables in the Magic Containers app settings
4. Push to `main` — the workflow handles the rest

### Docker (other platforms)

To build and run locally or deploy to any Docker-compatible platform:

```bash
docker build -t quickburrow .
docker run -p 3000:3000 quickburrow
```

Works with AWS ECS, Google Cloud Run, Azure Container Apps, Digital Ocean App Platform, Fly.io, Railway, etc.

### DIY Deployment

If you're familiar with deploying Node applications, the built-in app server is production-ready.

Make sure to deploy the output of `npm run build`

```
├── package.json
├── package-lock.json
├── build/
│   ├── client/    # Static assets
│   └── server/    # Server-side code
```

## Styling

This template comes with [Tailwind CSS](https://tailwindcss.com/) already configured for a simple default starting experience. You can use whatever CSS framework you prefer.

---

Built with ❤️ using React Router.
