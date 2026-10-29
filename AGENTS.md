# AGENTS.md

Persistent instructions for anyone implementing RIBH. Read this file and the docs in `docs/` before changing code.

## Product

RIBH is a premium SaaS product for Egyptian e-commerce businesses. Arabic is the primary language (RTL). English must work correctly in LTR. Currency formatting must support EGP.

Long-term: a secure, maintainable, multi-tenant SaaS. Near-term: frontend first, with architecture that can connect to Supabase later without a rewrite.

Canonical product, architecture, design, and sequencing documents:

- `docs/PRODUCT_BRIEF.md`
- `docs/ARCHITECTURE.md`
- `docs/DESIGN_SYSTEM.md`
- `docs/IMPLEMENTATION_ROADMAP.md`

Do not contradict those documents. If a decision must change, update the relevant doc in the same change.

## Scope control

- Implement one roadmap task at a time. Do not start the next task until the current task is complete and reviewable.
- Do not expand scope, add features, or "while we are here" refactors.
- Do not scaffold, install, or implement beyond the current task.
- Keep implementation prompts and task write-ups under 10,000 characters.
- Prefer simple, maintainable code over extra abstraction.

## Native project rules

This environment already applies platform-level agent rules. Do not duplicate them here.

Project-specific rules belong in this `AGENTS.md`. Do not add parallel rule files (`.cursorrules`, extra CLAUDE.md copies, or generated rule packs) unless a tool requires a file we do not already have.

## Code quality

- TypeScript strict mode. No `any` unless a documented escape hatch is unavoidable, and then isolate it.
- One clear responsibility per module. Name files and exports for the domain (orders, returns, profit), not for technical layers alone.
- Follow existing patterns once they exist. Do not introduce a second way to do the same thing.
- No leftover debug logging, commented-out code, or unused exports.
- Do not invent customers, integrations, performance numbers, or test results.

## Design consistency

- Marketing site and application dashboard share one design system. See `docs/DESIGN_SYSTEM.md`.
- Use design tokens (CSS variables / Tailwind theme), not ad-hoc hex values in components.
- Refined typography, disciplined spacing, accessible contrast, restrained color, clean tables, purposeful charts.
- Avoid generic AI-dashboard styling, decorative cards, oversized type in data views, random icons, visual clutter, and unnecessary animation.

## Accessibility

- WCAG 2.2 AA as the default bar: contrast, focus rings, labels, keyboard access, heading order.
- Visible labels on form fields. Do not use placeholder-only labels.
- Icon-only controls need an accessible name.
- Do not convey meaning by color alone.
- Honor `prefers-reduced-motion`.
- Touch targets at least 44px on interactive controls.

## RTL / LTR and locale

- Arabic (`ar`) is default. English (`en`) is fully supported.
- Set document `lang` and `dir` from the active locale (`rtl` for `ar`, `ltr` for `en`).
- Use CSS logical properties (`margin-inline-start`, `ps`/`pe`, `start`/`end`) instead of left/right.
- Format money with `Intl.NumberFormat` and currency `EGP`. Format dates and numbers with the active locale.
- Do not hard-code Latin-only UI copy in shared components.

## Security

- Never commit secrets, API keys, or `.env` files with real values. Use placeholders only.
- Do not read or copy LLM keys from the agent environment into this project.
- Do not log passwords, tokens, or full customer contact records.
- Treat future multi-tenancy as a hard boundary: no query or UI that assumes a single global data set without a tenant context.
- Validate and sanitize user input at the boundary once persistence exists. Until then, keep mock writes explicit and local.

## Architecture discipline

- UI components do not talk to a vendor SDK directly. They talk to domain types and data-access interfaces.
- Put business rules (order status transitions, net profit calculation) in domain modules, not in React components.
- Frontend may use mock repositories. Replacing mocks with Supabase must not require rewriting pages.

## Testing and verification

- Never claim something works unless it was verified with a command or a test.
- Domain logic gets unit tests (profit math, status transitions, money formatting).
- After scaffolding exists, run the project's lint and typecheck commands before calling a task complete.
- If those commands are not defined yet, say so. Do not invent them.
- Record the exact command and outcome in the task report.

## Verification checklist for every task

1. Scope matches the current roadmap task only.
2. Docs still agree with the code (update docs if a decision changed).
3. Lint/typecheck/tests run when they exist.
4. Arabic RTL and English LTR considered if UI changed.
5. No secrets, no unused files, no speculative features.
