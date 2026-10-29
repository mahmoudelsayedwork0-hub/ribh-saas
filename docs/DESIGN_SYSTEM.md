# RIBH Design System

Status: **proposal**. This is not a finished visual identity. Tokens, typefaces, and color values must be confirmed in a dedicated UI foundation task before they are treated as locked.

Quality reference: the restraint, typography, spacing, and statefulness of excellent consumer products (Apple's ecosystem as a quality bar, not a visual template to copy).

## Product feeling

RIBH should feel like a quiet, precise atelier for running a store: warm paper, dark ink, one accent used sparingly, numbers you can trust.

It is an operations product that also has a public marketing site. Both surfaces share tokens, type, components, and motion rules. The marketing site may use more whitespace and larger display type. The dashboard must stay denser, calmer, and numerically clear.

### Keywords

Warm, ink, limestone, restrained, bilingual, tabular, editorial-operations.

### Anti-goals

- Generic AI / indigo SaaS dashboards
- Crypto gold-on-black
- Excessive gradients, glassmorphism, neon accents
- Decorative cards with no action
- Oversized display type in tables and KPI rows
- Random icon styles or emoji-as-icons
- Visual clutter and motion that does not explain a state change

## Color strategy (proposal)

Direction: warm stone neutrals plus deep ink. One muted olive for profit/success. Semantic red and amber for danger and warning only.

Do not use bright purple, electric blue, or gold as brand color.

Proposed light theme (all hex values are **candidates** for contrast checking, not approved paint):

| Token | Candidate | Role |
|-------|-----------|------|
| `--background` | `#F6F3EE` | Page canvas (limestone) |
| `--foreground` | `#1A1916` | Primary text (ink) |
| `--card` | `#FFFCF8` | Raised surface |
| `--muted` | `#EBE6DC` | Subtle fills |
| `--muted-foreground` | `#6F6B63` | Secondary text (must still pass 4.5:1 on background) |
| `--border` | `#DDD6C8` | Hairline rules |
| `--primary` | `#1A1916` | Primary buttons, key marks |
| `--primary-foreground` | `#F6F3EE` | Text on primary |
| `--accent` | `#3F5C4A` | Profit, constructive emphasis (olive) |
| `--accent-foreground` | `#F6F3EE` | Text on accent |
| `--destructive` | `#B42318` | Destructive actions, failed delivery |
| `--warning` | `#9A6700` | Pending, attention |
| `--ring` | `#1A1916` | Focus ring |

Dark theme is allowed later as a full second set of tokens, not an inversion. Do not ship dark mode in the first UI tasks unless contrast for both themes is verified.

Rules:

- Components consume tokens, never raw hex.
- Profit green/olive is not the primary brand. Ink on limestone is the brand.
- Status colors need a text or icon companion. Color is not the only signal.
- Recheck every pair for WCAG 2.2 AA (`4.5:1` normal text, `3:1` large text and UI glyphs) before locking tokens.

## Typography (proposal)

Arabic is the default script. Any font without a complete Arabic set is rejected for UI use.

**Do not use Calistoga, Inter-only, or Latin display faces as the product UI font.** They fail Arabic.

Proposed pairing (to verify in the foundation task with real Arabic order copy):

| Role | Proposal | Fallback |
|------|----------|----------|
| UI Arabic | IBM Plex Sans Arabic | Noto Sans Arabic |
| UI Latin | IBM Plex Sans | Source Sans 3 |
| Tabular numbers | IBM Plex Sans tabular figures | `font-variant-numeric: tabular-nums` |
| Marketing display | Same family at larger size and tracked headings | No second display face until Arabic proof is clean |

Why this pairing: one family across Arabic and Latin, a serious tone, good figures for money, no costume "Oriental" display type.

Scale (proposal, rem at 16px root):

| Name | Size | Line height | Use |
|------|------|-------------|-----|
| `display` | 2.25rem (marketing only) | 1.2 | Marketing hero |
| `title` | 1.5rem | 1.3 | Page titles |
| `heading` | 1.125rem | 1.35 | Section headings |
| `body` | 1rem | 1.5 | Body, form fields |
| `caption` | 0.875rem | 1.45 | Helper text, table meta |
| `data` | 0.875rem | 1.4 | Tables; never below 0.75rem |

Weights: 400 body, 500 labels and nav, 600 titles. Avoid 900.

Marketing may use the display size. Dashboard page titles stop at `title`. KPI numbers may be large but not marketing-display large, and must use tabular figures.

Load fonts with `font-display: swap`, subset Arabic + Latin, and do not load unused weights.

## Spacing and layout

- 4px base grid; practical scale: 4, 8, 12, 16, 24, 32, 48, 64.
- Dashboard density is medium: information-first, not cramped.
- Marketing density is airier: more section padding, same tokens.
- One primary action per view.
- Max content width for reading: ~65ch. Data tables may use the full app canvas.
- Breakpoints: 375, 768, 1024, 1440. Mobile-first.
- No horizontal page scroll. Tables wrap in `overflow-x-auto` or collapse to cards below 768.

## Component standards

Build on shadcn/ui primitives, then skin them with RIBH tokens. Do not restyle each screen independently.

| Component | Standard |
|-----------|----------|
| Buttons | One primary, one secondary, one destructive. Height >= 40px (44px on mobile). Visible focus. Loading replaces the label, button disabled. |
| Inputs | Visible label, helper text, error text under the field. No placeholder-only labels. Height >= 44px on mobile. |
| Tables | Hairline borders or row rules, not boxed-card-per-row. Tabular numbers, right/start aligned money columns per locale. Sortable headers when sorting exists. |
| Cards | Use only to group a real section. No stacked metric cards with icons for decoration. |
| Navigation | Desktop: sidebar. Mobile: bottom bar or a single menu, max 5 top-level items. Icon + label. Active state by weight and a mark, not color alone. |
| Dialogs | Esc and explicit close. Confirm before destructive. Unsaved-change guard when a form is dirty. |
| Empty states | One sentence + one action. No illustrations required for MVP. |
| Toasts | Short, 3–5s, `aria-live="polite"`, do not steal focus. |
| Charts | Later, and only for trends that a table cannot show. Line for profit over time, bar for status mix. Always offer a numeric summary. No pie charts in MVP. |

Icons: Lucide only, consistent stroke, 20–24px. No emoji icons.

## Motion

- Default: 150–200ms opacity/transform only.
- No layout animation (width/height/top/left).
- No page-load choreography.
- `prefers-reduced-motion: reduce` disables non-essential motion.

## Arabic RTL / English LTR

- `html[lang="ar"]` has `dir="rtl"`. `html[lang="en"]` has `dir="ltr"`.
- Use logical CSS and Tailwind logical utilities (`ms`, `me`, `ps`, `pe`, `start`, `end`, `text-start`).
- Mirroring: chevrons, sidebars, and filters flip; logos and numbers do not reverse glyph-by-glyph.
- Phone numbers and order IDs stay in Latin digits unless a later localization task decides otherwise. Money uses locale-aware `Intl` output.
- Do not ship separate "Arabic theme" colors. Direction changes; brand does not.

## Accessibility requirements

- WCAG 2.2 AA.
- Keyboard: all actions reachable; focus visible; no trap.
- Forms: `label` + `for`, errors associated with `aria-describedby`, first invalid field focused on failed submit.
- Contrast as above.
- Skip link to main content on the app shell.
- Charts and status dots have text equivalents.

## Responsive behavior

| Surface | Phone | Desktop |
|---------|-------|---------|
| Marketing | Single column, stacked CTA | Generous margins, two-column feature rows allowed |
| App nav | Bottom or top sheet | Persistent sidebar |
| Orders | Card list | Data table |
| Order form | One column | Two-column grouping (customer / items) |
| Profit | Stacked summary, then table | Summary row + table; chart only if it still reads at 375px |

Primary operations target is desktop. Phone must still complete status updates and order lookup.

## Shared system for marketing and app

Same: color tokens, type family, buttons, inputs, radii, motion, icon set.

Different: spacing density, maximum heading size, navigation chrome.

Radius proposal: 8px controls, 12px containers. Not pill-shaped by default.

Shadow proposal: almost none. Separation by border and background, not elevation stacks.

## Open questions (resolve in the UI foundation task)

1. Lock IBM Plex Sans Arabic vs Noto Sans Arabic after rendering real order tables.
2. Confirm olive `#3F5C4A` vs a cooler ink-only palette (no chromatic accent).
3. Decide whether marketing uses a true display size or stays in the app type scale.
4. Defer dark mode.

Until those are decided in implementation, treat this file as the intent document, not a Figma spec.
