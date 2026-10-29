# RIBH Architecture

Status: recommended initial architecture. No application has been scaffolded yet.

## Current repository and environment

Inspected 2026-10-29.

| Item | Finding |
|------|---------|
| GitHub remote | `https://github.com/mahmoudelsayedwork0-hub/ribh-saas` |
| Branch | `main` at `dc0f8a8` ("Initial commit") |
| Working tree | Essentially empty: `README.md` contains only `# ribh-saas` |
| Application code | None |
| `package.json` / lockfile | None |
| Docker | Not available |
| Node | v22.22.0 |
| npm | 10.9.4 |
| Yarn | 1.22.22 (Classic) |
| pnpm / bun | Not installed |
| TypeScript compiler | Not installed globally |
| Python | 3.11.2 |
| Go | 1.25.6 |
| Locale | `C.UTF-8` (no Arabic system locale; browsers/Intl still support `ar-EG`) |
| Memory | ~7.8 GiB RAM on the host; available headroom can be tight |
| Disk | ~17 GiB free |

Implications:

- The project starts from a blank Node-capable environment, not from an existing app.
- Prefer **npm** as the package manager (already current; avoids Yarn Classic quirks and a pnpm install).
- Do not assume Dockerized Postgres or a local Supabase stack.
- Next.js is compatible with Node 22. Keep the dependency set small; this host is not a large CI runner.

## Recommended stack (proposal)

These are proposals. Packages must not be installed until the matching roadmap task.

| Layer | Proposal | Why | Tradeoff |
|-------|----------|-----|----------|
| Framework | Next.js App Router | One codebase for marketing + app; locale routing; later Server Components and Route Handlers for Supabase | Heavier than Vite; higher memory during `next dev` |
| Language | TypeScript, `strict` | Safer domain math and refactors | Slightly more upfront typing |
| Styling | Tailwind CSS | Token-friendly, RTL via logical utilities | Must forbid ad-hoc values |
| UI primitives | shadcn/ui | Copy-in components we own; matches Tailwind | Coupled to its patterns; pin versions at add time |
| Icons | Lucide | One consistent stroke family | Do not mix other icon sets |
| i18n | `next-intl` (evaluate at install) | App Router + locale prefixes + messages | Adds routing convention; choose once |
| Unit tests | Vitest (when tests start) | Fast for domain logic | Not installed yet |
| Persistence | Supabase + PostgreSQL **later** | Auth + RLS + hosted Postgres without Docker here | Not for frontend tasks |
| Charts | Recharts or similar **only when** a profit chart task starts | Keep charts out of the foundation | Do not add a chart library speculatively |

### Alternatives considered

1. **Vite + React Router.** Faster local startup, less memory. Marketing SEO and future auth/session work become a second project or a bolted-on SSR story. Higher rewrite risk when adding Supabase SSR. Not recommended for this product.
2. **Remix.** Strong form model, smaller mindshare with shadcn recipes and Supabase SSR examples than Next.js. Not recommended unless Next.js is rejected after a scaffold spike.
3. **Separate marketing site and app.** Clean split, twice the design-system wiring. Rejected: the brief requires one design system and a small team.

If Next.js cannot run in this environment during Task 01, stop and record the failure. Do not silently switch stacks.

## Architectural principles

1. **Frontend first, backend-ready.** Pages render from repository interfaces. First implementations may be in-memory mocks.
2. **Domain in the middle.** Order status rules and net profit live in pure functions. React components do not own those rules.
3. **One design system.** Marketing and app import the same tokens and primitives.
4. **Locale is a first-class boundary.** `lang`, `dir`, copy, and number format change together.
5. **No vendor lock-in in UI.** No Supabase client inside components. No courier SDK in pages.
6. **Small surface.** Do not add API routes, middleware auth, or database clients before their roadmap tasks.

## Proposed application shape

The separation below is the decision to preserve. Folders are added in the matching roadmap task, not all during Task 01. Names may shift slightly at first use; the boundaries must not.

```
app/
  [locale]/
    (marketing)/          # public site
    (app)/                # authenticated-looking app shell (no real auth yet)
      orders/
      returns/
      profit/
      settings/
components/
  ui/                     # shadcn primitives
  layout/                 # shells, nav, page header
  marketing/
  orders/
  returns/
  profit/
domain/
  orders/                 # types, status transitions
  returns/
  profit/                 # net profit formula
  money/                  # EGP formatting helpers
data/
  types.ts                # shared DTO types if needed
  repositories/           # interfaces only
  mocks/                  # in-memory implementations
  supabase/               # empty until backend tasks
lib/
  i18n/
  cn.ts
messages/
  ar.json
  en.json
```

Route groups keep marketing and application layouts separate while sharing tokens.

### Routes vs components vs domain vs data

| Kind | Owns | Must not own |
|------|------|----------------|
| Routes (`app/`) | Composition, loading/error boundaries, locale | Profit formulas, direct `fetch` to Supabase |
| Components | Rendering, local UI state, accessibility | Persistence, cross-page business rules |
| Domain | Types, transitions, calculations, invariants | JSX, HTTP, environment variables |
| Data-access interfaces | Method contracts (`listOrders`, `getOrder`, `saveOrder`) | UI concerns |
| Mock repositories | In-memory data that satisfies interfaces | Secret production behavior |
| Future Supabase repositories | Mapping tables <-> domain types, auth-aware queries | Styling, routing |

## Data-access interfaces (so Supabase does not force a rewrite)

Define TypeScript interfaces before any UI data loading pattern hardens. Example shape (illustrative, not implemented):

```ts
export interface OrderRepository {
  list(query: OrderListQuery): Promise<Order[]>;
  getById(id: OrderId): Promise<Order | null>;
  save(order: Order): Promise<Order>;
}
```

Rules:

- Pages and hooks depend on the interface, not on the mock module name leaking everywhere. A thin factory (`getRepositories()`) is enough.
- Mock data is explicit sample data, not fake analytics.
- When Supabase arrives, add `data/supabase/order-repository.ts` that implements the same interface. Swap the factory. Keep domain types stable.
- Database columns may differ from domain fields. Mapping happens in the Supabase adapter, not in components.

Auth, when added, should wrap the app route group (session in server context) and pass `storeId` / `tenantId` into repositories. UI should already thread a store context object even if it is a constant mock (`store-1`) in the frontend phase.

## Future backend services

Not in the frontend phase. Planned so we do not block them:

| Service | Role | When |
|---------|------|------|
| Supabase Auth | Email/password or magic link for the store owner | After the mock app is usable |
| PostgreSQL via Supabase | Orders, order lines, returns, store profile | With Auth |
| Row Level Security | Tenant isolation | With the first persisted table |
| Storage | Optional later (waybill images) | Not MVP |
| Edge / Route Handlers | Webhooks from couriers | After a real integration is validated |

No Redis, message bus, microservices, or custom Nest/Go API is proposed. A single Next.js app plus Supabase is enough until proven otherwise.

## i18n and direction

- Locale prefix in the URL (`/ar/...`, `/en/...`) so each screen is deep-linkable in both languages.
- Default locale: `ar`.
- `dir="rtl"` for Arabic, `dir="ltr"` for English, set on `<html>`.
- Messages in JSON files, not inline Arabic/English forks of components.
- Money and dates go through `domain/money` and `Intl`, never through concatenated strings.

## Security posture (even before backend)

- No secrets in the repo.
- Mock repositories are not an excuse to render unbounded PII in logs.
- When Supabase is added: server-side client for privileged work, user-scoped client in the browser, RLS on every table, no service role key in client bundles.
- Multi-tenant: every persisted row will need `store_id` (name TBD at schema time). Frontend types should include `storeId` from the start.

## Testing shape

| Kind | What | When it starts |
|------|------|----------------|
| Unit (Vitest) | Profit formula, status transitions, money format | First task that adds that domain module |
| Typecheck | `tsc --noEmit` | From scaffold onward |
| Lint | ESLint via Next.js defaults | From scaffold onward |
| E2E | Not in the first tasks | Only after a stable app shell exists |

Do not add Cypress and Playwright together. Choose later, once there is UI worth driving.

## What this architecture deliberately postpones

- Auth middleware and protected routes (UI may *look* like an app without locking it).
- Real environment variables for Supabase.
- API versioning.
- Background jobs.
- Observability stacks.

Those wait for backend tasks in `docs/IMPLEMENTATION_ROADMAP.md`.
