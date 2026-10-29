# RIBH Implementation Roadmap

Status: sequential plan. Complete one task, make it reviewable, then start the next. Do not implement later tasks early.

Source of truth for product, stack, and visual intent:

- `docs/PRODUCT_BRIEF.md`
- `docs/ARCHITECTURE.md`
- `docs/DESIGN_SYSTEM.md`
- `AGENTS.md`

Keep every future implementation prompt under 10,000 characters. Do not delegate parallel implementation tasks.

## How to use this document

- **Frontend phase first.** Backend / Supabase tasks start only after Task 16 is complete and reviewable, unless a later decision explicitly changes that.
- Each task is independently reviewable. A reviewer should be able to reject one task without needing the next.
- Do not install packages, scaffold, or add files outside the current task's scope.
- Verification must use real commands. If a command is not defined yet, say so.
- Sample data in mocks is labeled as sample data. Do not present it as real customers.

## Phase map

| Phase | Tasks | Outcome |
|-------|-------|---------|
| 0. Specify | 00 | Docs only (this phase) |
| 1. Foundation | 01–03 | Runnable app, tokens, locales |
| 2. Domain | 04–07 | Types, rules, mocks, no full UI yet |
| 3. Shell and marketing | 08–09 | Shared chrome + public site |
| 4. Operations UI | 10–15 | Orders, returns, profit, settings |
| 5. Frontend close | 16 | Accessibility and consistency pass |
| 6. Backend later | 17+ | Auth and persistence — not before Task 16 |

---

## Task 00: Repository discovery and project specification

**Status:** complete (documentation only). Application code was not implemented.

**Objective:** Inspect the empty repository and environment; write the canonical docs.

**Scope:** `AGENTS.md`, `docs/PRODUCT_BRIEF.md`, `docs/ARCHITECTURE.md`, `docs/DESIGN_SYSTEM.md`, `docs/IMPLEMENTATION_ROADMAP.md`. No application code.

**Dependencies:** None.

**Acceptance criteria:**

- Repository contents and tooling are recorded.
- The five documents agree with each other.
- No scaffold, no installs, no UI.

**Verification:** File list of the repo; read-through for contradictions; confirm no `package.json` was added.

---

## Task 01: Next.js application scaffold

**Objective:** Create a runnable Next.js App Router project with TypeScript strict mode, Tailwind CSS, and npm, without product UI.

**Scope:**

- Scaffold Next.js (App Router) in the repository root using npm.
- TypeScript `strict`.
- Tailwind CSS.
- ESLint via Next.js defaults.
- `.gitignore` appropriate for Next.js.
- `allowedHosts` for this environment's preview domain if the dev server requires it (see platform Vite/Next host rules).
- A single placeholder home page proving the app boots.
- Record exact `package.json` scripts: at minimum `dev`, `build`, `lint`, and a typecheck path (`tsc --noEmit` or `next build`).

**Out of scope:** shadcn, i18n, design tokens, domain code, marketing content, auth, Supabase, extra libraries (no chart lib, no state lib).

**Dependencies:** Task 00.

**Acceptance criteria:**

- `npm install` completes in this environment.
- `npx tsc --noEmit` (or the documented equivalent) exits 0.
- `npm run lint` exits 0.
- `npm run build` exits 0, or a documented, observed failure is reported and the task stops for a stack decision.
- No product features.

**Verification:** Run `npm run lint`, typecheck, and `npm run build`. Record commands and exit codes. Do not claim the app "looks right" beyond a successful boot/build.

**Environment note:** This host has limited free RAM. If `next dev` or `next build` is killed, report it. Do not silently switch to Vite.

---

## Task 02: Design tokens and shadcn primitives

**Objective:** Encode the proposed design system as CSS variables / Tailwind theme and add a minimal owned primitive set.

**Scope:**

- CSS variables from `docs/DESIGN_SYSTEM.md` (light theme only).
- Tailwind theme mapping to those tokens.
- Add shadcn/ui in a pinned, documented way.
- Install Lucide.
- Add only the primitives required to prove the system: button, input, label. No extra components "for later".
- A private `/ui-preview` (or similar) route that shows tokens, type scale, and the three primitives in Arabic placeholder copy. Not linked from marketing.

**Out of scope:** Full marketing page, app shell, dark mode, charts, custom illustrations.

**Dependencies:** Task 01.

**Acceptance criteria:**

- No raw hex in the three primitives; they use tokens.
- Focus rings are visible.
- Contrast of text/background pairs is checked against the AA rule and documented if a candidate token fails (adjust token, do not ignore).
- Lint and typecheck pass.

**Verification:** `npm run lint` and typecheck. Manual check of `/ui-preview` at 375px and 1024px if a preview server is available; otherwise screenshot-free description of what was built and which command passed.

---

## Task 03: Locales, direction, and message files

**Objective:** Make Arabic the default locale with RTL, and English a full LTR locale, at the routing and document level.

**Scope:**

- Choose and install one i18n approach (proposal: `next-intl`); document the choice in `docs/ARCHITECTURE.md` if it differs.
- Locale-prefixed routes: `/ar`, `/en`.
- `<html lang>` and `dir` derived from locale.
- `messages/ar.json` and `messages/en.json` with only strings needed for the placeholder home and ui-preview.
- Locale switch control with an accessible name.
- Logical CSS on anything touched.

**Out of scope:** Translating a full product, date picker libraries, storing locale in a database.

**Dependencies:** Task 02.

**Acceptance criteria:**

- Visiting the default URL lands on Arabic, `dir="rtl"`, `lang="ar"`.
- `/en` is `dir="ltr"`, `lang="en"`.
- Switcher changes locale without breaking layout.
- Lint and typecheck pass.

**Verification:** Lint/typecheck. Confirm both URLs render (dev server or build + start). Record how default locale is configured.

---

## Task 04: Money, dates, and shared domain types

**Objective:** Add testable formatting and core IDs/types used by later domain modules.

**Scope:**

- `domain/money` (or equivalent): format EGP with `Intl.NumberFormat`, locale-aware.
- Date formatting helper using `Intl` and the active locale.
- Shared branded IDs / types: `StoreId`, `OrderId`, etc., with `storeId` present from the start.
- Vitest (or the test runner chosen in this task) added because this is the first unit-tested domain code.
- Unit tests for money formatting in `ar-EG` and `en-EG` (or `en` with currency EGP).

**Out of scope:** Order workflow, UI tables, rounding policy beyond what tests specify. If a rounding rule is needed, document it in the product brief in the same change.

**Dependencies:** Task 03 (locale exists). If adding Vitest is easier before i18n, still do not skip locale-aware tests.

**Acceptance criteria:**

- Tests cover at least: integer pounds, piastres if supported, negative amounts, zero.
- No hardcoded `"EGP 100"` string concatenation in the helper.
- Lint, typecheck, and unit tests pass.

**Verification:** Exact test command (e.g. `npx vitest run`) plus lint/typecheck.

---

## Task 05: Order model and status transitions

**Objective:** Define the order domain and which status changes are legal.

**Scope:**

- Order and order-line types matching the MVP fields in the product brief.
- Status union: at least `new`, `confirmed`, `shipped`, `delivered`, `returned`, `cancelled`.
- Pure functions: `canTransition`, `transition`.
- Unit tests for legal and illegal transitions (e.g. delivered -> cancelled is rejected unless explicitly allowed and documented).

**Out of scope:** UI, persistence, courier events, inventory.

**Dependencies:** Task 04.

**Acceptance criteria:**

- Transitions live in domain code, not in a component.
- Tests fail closed: unknown transitions throw or return a typed error, not silent no-ops.
- Lint, typecheck, tests pass.

**Verification:** Unit tests for the transition table; lint/typecheck.

---

## Task 06: Return model and net profit calculation

**Objective:** Encode the working net-profit formula and return impact as pure functions.

**Scope:**

- Types for a return against an order.
- `calculateOrderNetProfit(order, context)` implementing the formula in `docs/PRODUCT_BRIEF.md`.
- Period total helper: sum of in-scope orders.
- Unit tests: delivered profitable order; cancelled before shipping; returned after shipping (fees remain, collected amount zeroed); discount; zero costs.

**Out of scope:** UI charts, ads spend, inventory shrinkage, tax.

**Dependencies:** Task 05.

**Acceptance criteria:**

- Formula matches the product brief; if the implementation must differ, the brief is updated in the same change.
- Components are not added in this task.
- Lint, typecheck, tests pass.

**Verification:** `npx vitest run` (or documented command) targeting profit tests.

---

## Task 07: Data-access interfaces and mock repositories

**Objective:** Introduce repository interfaces and in-memory mocks so UI tasks never import a future Supabase client.

**Scope:**

- Interfaces: orders, returns, store profile.
- Mock implementations with a small, clearly fake sample dataset (e.g. "Sample Store", fictional names labeled as samples).
- Factory `getRepositories()` returning mocks.
- Optional thin tests that the mock round-trips `save` / `list`.

**Out of scope:** Supabase, network, UI.

**Dependencies:** Task 06.

**Acceptance criteria:**

- No page would need to change its import of the *interface* when mocks are swapped (factory is the only expected change later).
- Sample data is obviously sample data.
- Lint, typecheck, and any new tests pass.

**Verification:** Lint/typecheck; mock unit tests if added.

---

## Task 08: Application shell

**Objective:** Provide the daily-use chrome: navigation, page header, locale switch, responsive layout.

**Scope:**

- App route group layout.
- Desktop sidebar and mobile navigation, max five top-level items: Orders, Returns, Profit, Settings, Home (marketing).
- Skip link, main landmark, accessible nav labels.
- Empty placeholder pages for the four app areas (title only).

**Out of scope:** Real tables, forms, charts, auth gating.

**Dependencies:** Task 07 (so later pages can plug in) and Task 03.

**Acceptance criteria:**

- Arabic shell is RTL; English is LTR.
- Nav works with keyboard.
- Touch targets on mobile nav meet 44px.
- Lint and typecheck pass.

**Verification:** Lint/typecheck. Keyboard tab through nav in both locales if preview is available.

---

## Task 09: Marketing site (static)

**Objective:** Ship a small public site that explains RIBH using the same design system, without fake metrics or invented customers.

**Scope:**

- Marketing layout distinct from the app shell.
- Sections: product explanation, who it is for, what the MVP does, CTA to the app (mock) and locale switch.
- Copy in `ar` and `en`. Honest: no testimonials, no unverified numbers, no fake logos.

**Out of scope:** Blog, pricing experiments, CMS, analytics pixels.

**Dependencies:** Task 08 (shared primitives and locales). May depend only on 02–03 if shell and marketing need to land in either order; prefer shell first so CTA has a real destination.

**Acceptance criteria:**

- Same tokens as the app.
- RTL/LTR correct.
- No fabricated social proof.
- Lint and typecheck pass.

**Verification:** Lint/typecheck. Read copy against the product brief assumptions table (must not present assumptions as facts).

---

## Task 10: Orders list

**Objective:** List sample orders with status, customer, total, and net profit column.

**Scope:**

- Orders index page using the order repository.
- Desktop table, mobile card list.
- Filter by status.
- Accessible table headers; money via domain formatter.

**Out of scope:** Create form, bulk actions, export, pagination beyond a simple in-memory list.

**Dependencies:** Task 08, Task 07.

**Acceptance criteria:**

- Uses repository + domain profit helper, not inline math in JSX.
- Filter does not rely on color alone.
- Lint/typecheck pass.

**Verification:** Lint/typecheck. Confirm sample rows render for `ar` and `en`.

---

## Task 11: Create and edit order

**Objective:** Manual order entry that a seller could complete on phone or desktop.

**Scope:**

- Form: customer name, phone, address, line items (name, qty, unit price, product cost), shipping cost, COD fee, other fees, discount, notes.
- Visible labels, validation messages, submit loading state writing to the mock repository.
- After save, return to list or detail.

**Out of scope:** Autocomplete from WhatsApp, SKU catalog, barcode.

**Dependencies:** Task 10.

**Acceptance criteria:**

- Invalid submit focuses the first error.
- Values persist in the mock for the session (document whether mock is in-memory only).
- Lint/typecheck pass.

**Verification:** Lint/typecheck. Manual happy-path and one validation failure if preview exists.

---

## Task 12: Order detail and status workflow

**Objective:** Show one order and apply legal status transitions.

**Scope:**

- Detail page.
- Status actions from `canTransition`.
- Illegal actions not shown, or shown disabled with reason.
- Confirm destructive transitions (cancel).

**Out of scope:** Courier tracking numbers as a live integration; printing.

**Dependencies:** Task 11, Task 05.

**Acceptance criteria:**

- UI cannot apply a transition the domain rejects.
- Arabic and English both work.
- Lint/typecheck pass.

**Verification:** Lint/typecheck. Domain tests still pass. Manual: attempt an illegal transition if UI is previewable.

---

## Task 13: Returns

**Objective:** Record a return against an eligible order and see it on a returns list.

**Scope:**

- Returns list.
- Action from an eligible order (delivered or as defined in domain).
- Reason field, fee impact via existing profit function.
- Empty state.

**Out of scope:** Partial line returns unless the domain already supports them; warehouse restock.

**Dependencies:** Task 12, Task 06.

**Acceptance criteria:**

- Returning an order updates status through domain functions.
- Profit for that order matches the returned-order test cases.
- Lint/typecheck pass.

**Verification:** Lint/typecheck; profit unit tests still green.

---

## Task 14: Profit view

**Objective:** Show period net profit from orders, not a separate ledger.

**Scope:**

- Summary: collected, costs, net profit, order count, return count.
- Table of contributing orders.
- Period control: at least "all sample data" and one bounded range (e.g. this month) using locale dates.
- No chart required. If a single small line chart is added, it must have a numeric summary and no extra library unless necessary; prefer no chart in this task.

**Out of scope:** Year-over-year, advertising ROI, CSV export, pie charts.

**Dependencies:** Task 13.

**Acceptance criteria:**

- Totals equal the domain period helper for the same inputs.
- Do not invent trend percentages.
- Lint/typecheck pass.

**Verification:** Compare one fixture: helper total vs UI total (unit test of helper is mandatory; UI check if previewable). Lint/typecheck.

---

## Task 15: Store settings (profile and locale)

**Objective:** Let the operator set store name and preferred language; currency remains EGP.

**Scope:**

- Settings page bound to store profile repository.
- Store name, default locale.
- Currency display is EGP and not user-switchable in MVP (explain in helper text).

**Out of scope:** Team members, billing, API keys, logo upload.

**Dependencies:** Task 08, Task 07.

**Acceptance criteria:**

- Name change reflects in the shell header.
- Locale preference uses existing i18n, does not fork a second mechanism.
- Lint/typecheck pass.

**Verification:** Lint/typecheck. Manual save round-trip if previewable.

---

## Task 16: Frontend quality pass

**Objective:** Close the frontend phase with accessibility, RTL, and consistency fixes. No new features.

**Scope:**

- Keyboard pass on marketing, list, form, detail, settings.
- Contrast spot-check of tokens in use.
- `prefers-reduced-motion`.
- Remove unused components/exports from earlier tasks.
- Update docs if implemented behavior diverged.

**Out of scope:** New pages, backend, extra animations.

**Dependencies:** Tasks 09–15.

**Acceptance criteria:**

- No known WCAG-blocking issues introduced by MVP screens (document residual issues honestly).
- Lint, typecheck, unit tests all pass.

**Verification:** Full frontend verification set: lint, typecheck, unit tests. List residual issues rather than claiming perfection.

---

## Backend phase (do not start before Task 16)

The following tasks exist so frontend work does not paint the project into a corner. They are **not** the next task after 00.

### Task 17: Supabase project wiring (no tables yet)

Env placeholders, server/client boundary, no secrets committed. Verification: app still builds with placeholders.

### Task 18: Auth (email) and session-gated app routes

Replace the open app shell with real session checks. Verification: unauthenticated users cannot load app routes.

### Task 19: Postgres schema for store, orders, lines, returns + RLS

`store_id` on every row. Verification: documented SQL and RLS policies; no UI rewrite beyond the repository factory.

### Task 20: Supabase order/return/store repositories

Implement interfaces from Task 07. Drop mocks from production factory; keep mocks for tests. Verification: integration tests or a recorded manual persist check; frontend pages unchanged except factory.

Further backend tasks (billing, courier webhooks, file storage) are intentionally unspecified until customer validation of A2, A6, and A10 in the product brief.

---

## Explicitly not scheduled

- Courier APIs
- WhatsApp ingestion
- Inventory
- Native apps
- Dark mode
- Microservices
- Parallel implementation tracks

---

## Next task after this document is reviewable

**Task 01: Next.js application scaffold.**

Do not begin Task 01 until Task 00 is complete and its results are reviewable.
