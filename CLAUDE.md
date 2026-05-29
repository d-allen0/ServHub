---

## Key Rules — Read Before Every Task

1. **RLS is non-negotiable.** Every new table gets an `org_id` column and an
   isolation policy before any other work. Never rely on UI-level filtering alone.

2. **org_id always comes from the JWT, never from the request body.**

3. **Terminology is always dynamic.** Never hardcode "Job", "Worker", "Quote",
   or "Client" in any user-facing string. Always use `useTerminology()` (web)
   or `OrgTheme.label()` (iOS).

4. **Storage paths are always org-scoped.** All uploads go under `orgs/[org_id]/...`

5. **Credentials live in Supabase Vault.** Never in `.env` files or plaintext columns.

6. **TypeScript strict mode is always on.** No `any`. No unexplained non-null assertions.

7. **iPhone app is offline-first.** Cache jobs in SwiftData. Queue updates locally.

8. **Test with all 3 roles after every feature.** Not done until all three roles verified.

9. **Activity log every write.** Every create/update/delete on core entities logs to `activity_log`.

10. **Never break another org.** Migrations must be backward-compatible.

11. **Payment providers are per-org and pluggable.** Always route through `getPaymentService(org)`.

12. **QuickBooks is accounting sync, not a payment processor.**

13. **OAuth tokens for QuickBooks must be refreshed proactively.**

---

## Payment Provider Abstraction

```typescript
// /web/lib/payments/index.ts
export type PaymentProvider = 'stripe' | 'quickbooks' | 'square' | 'manual'

export interface PaymentService {
  createCheckoutSession(invoice: Invoice): Promise<{ url: string }>
  recordManualPayment(invoice: Invoice, amount: number, method: string): Promise<Payment>
  syncInvoice(invoice: Invoice): Promise<{ externalId: string }>
  handleWebhook(payload: unknown): Promise<void>
}

export async function getPaymentService(org: Org): Promise<PaymentService> {
  switch (org.payment_provider) {
    case 'stripe':      return new StripeService(org)
    case 'quickbooks':  return new QuickBooksService(org)
    case 'square':      return new SquareService(org)
    default:            return new ManualService(org)
  }
}
```

---

## Environment Variables (Web — `.env.local`)

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://plwgrhfwfemwstkgdmfc.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Google Maps
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=

# App domain
NEXT_PUBLIC_APP_DOMAIN=app.yourdomain.com

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# QuickBooks
QBO_CLIENT_ID=
QBO_CLIENT_SECRET=
QBO_REDIRECT_URI=
QBO_ENVIRONMENT=

# Square
SQUARE_APPLICATION_ID=
SQUARE_APPLICATION_SECRET=
SQUARE_ENVIRONMENT=
```

---

## Accounts to Set Up (Day 1 Checklist)

- [x] **Supabase** — project created; schema + RLS migrations applied (17 tables)
- [x] **GitHub** — repo initialized with `/web`, `/ios`, `/supabase`, `/fastlane` directories
- [x] **Node.js** — v24.16.0 installed
- [x] **VS Code** — v1.121.0 installed
- [x] **Claude Code** — installed via npm
- [ ] **Vercel** — connect GitHub repo; configure environment variables
- [ ] **Apple Developer Program** — $99/year; required for TestFlight + App Store
- [ ] **Stripe** — create platform account; enable Connect for per-org payouts
- [ ] **Intuit Developer** — create app; enable QuickBooks Online Accounting scope
- [ ] **Square Developer** — create app; note application ID + secret
- [ ] **Twilio** — create account; register 10DLC brand; get long code number
- [ ] **Google Cloud** — enable Maps JavaScript API; create restricted API key
- [ ] **Domain** — point `*.app.yourdomain.com` wildcard DNS to Vercel

---

## Development Phases

| Phase | Focus | Status |
|---|---|---|
| 1 | Multi-tenant auth, roles, org creation, invite flow | 🔄 In Progress |
| 2 | Core loop: clients → jobs → quotes → invoices → payments | ⏳ Pending |
| 3 | Field tools: photos, GPS time tracking, map view, route optimization | ⏳ Pending |
| 4 | Communication: push notifications, Twilio SMS, client hub portal | ⏳ Pending |
| 5 | Pluggable payments, reporting, activity log, branding settings | ⏳ Pending |
| 6 | White-label delivery: Fastlane, subdomain routing, org onboarding | ⏳ Pending |

### Phase 1 Progress
- [x] Supabase schema written and applied (17 tables, RLS enabled)
- [ ] Next.js project scaffolded in `/web`
- [ ] Supabase Auth configured (email invite flow)
- [ ] JWT custom claims set up (org_id + role in app_metadata)
- [ ] Login page built
- [ ] Role-based middleware and redirects
- [ ] Admin team management page (invite, assign role, deactivate)
- [ ] Test: all 3 roles verified, RLS confirmed blocking cross-org queries
