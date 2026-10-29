# RIBH Product Brief

Status: working brief for MVP planning. Not a substitute for customer interviews.

## Vision

RIBH helps Egyptian online sellers manage orders, track deliveries and returns, and understand their real net profit.

The product should feel calm, precise, and premium: an operations tool the owner opens every day, not a generic dashboard.

Long-term, RIBH is a secure, maintainable, multi-tenant SaaS. The first release is a focused operations surface for a single store, with room to add authentication, persistence, and tenancy without rewriting the product.

## Target customer

Small and medium Egyptian stores that sell through:

- Social media (Instagram, Facebook, TikTok)
- Messaging channels (WhatsApp, Messenger)
- Simple e-commerce websites

Cash on delivery (COD) is common. The owner often runs the store personally, sometimes with one or two helpers. They already take orders; they struggle to keep those orders, deliveries, returns, and costs in one trustworthy place.

RIBH is not aimed at large marketplaces, enterprise ERPs, or accounting firms in the first version.

## Problems we believe we are solving

These are working hypotheses until validated with real sellers.

1. **Orders live in too many places.** Screenshots, chat threads, spreadsheets, and memory do not agree.
2. **Delivery status is informal.** "Out for delivery" and "customer did not answer" are tracked by hand, if at all.
3. **Returns quietly destroy profit.** A returned COD order still incurred shipping, packaging, and time.
4. **Revenue is mistaken for profit.** Owners see collected cash and miss product cost, courier fees, COD fees, discounts, and failed deliveries.
5. **Tools are either too heavy or too ugly.** Spreadsheets are flexible but fragile. Many Arabic tools feel cluttered. Global tools ignore COD, Arabic, and EGP.

## Primary use cases (MVP)

A store owner should be able to:

1. **Record an order** with customer name, phone, address, items, selling price, and notes.
2. **See all open orders** in a scannable list, filtered by status.
3. **Move an order through delivery states** (for example: new, confirmed, shipped, delivered, returned, cancelled).
4. **Record a return** against an order and see that it affects profit.
5. **See net profit** for a period, derived from order economics, not from a separate accounting system.
6. **Use the product in Arabic (RTL)** as the default, and switch to English (LTR) when needed.
7. **See money in EGP** with correct locale formatting.

A visitor should be able to:

8. **Understand RIBH from a marketing site** that uses the same design system as the application.

## Initial MVP boundaries

### In scope for the first product slice

- Bilingual marketing site (static content).
- Application shell for daily use.
- Manual order create, read, update.
- Manual delivery status workflow.
- Manual returns against orders.
- Net profit calculated from order-level costs and outcomes.
- Store profile basics: store name, language, currency display (EGP).
- Mock or local data only, behind repository interfaces.

### Out of scope until later, validated work

- Real authentication and billing.
- PostgreSQL / Supabase persistence and multi-tenant isolation.
- Live courier integrations (Bosta, Mylerz, Aramex, and others).
- WhatsApp, Instagram, or Shopify/WooCommerce ingestion.
- Inventory, purchasing, or supplier management.
- Accounting, VAT returns, or bank reconciliation.
- Staff roles and permissions beyond a single operator.
- Native mobile apps.
- Notifications, SMS, and automated customer messages.
- Invented marketplace, courier, or payment capabilities.

Frontend is built first. Backend is added later. UI must not hard-wire a vendor SDK.

## Net profit (working definition)

Until a customer-validated cost model exists, net profit for an order is:

```
netProfit = collectedAmount
          - productCost
          - shippingCost
          - codFee
          - otherFees
          - discount
```

Rules for the first version:

- A cancelled order before shipping: `collectedAmount` is `0` and incurred costs are `0`, so `netProfit` is `0`.
- A returned order after a delivery attempt: `collectedAmount` is `0` (refunded or never kept). Shipping, COD, and other fees still count as incurred. Product cost still subtracts until inventory restock exists (out of scope).
- Totals on the profit view are the sum of in-scope orders for the selected period.
- This formula is a product decision, not an accounting standard. It must stay in domain code so it can change without rewriting UI.

Do not present this formula as legally or financially certified.

## Languages, money, and devices

- Arabic is primary. English is a first-class second locale, not an afterthought.
- Currency: EGP. Do not default to USD.
- Experience must work on a laptop (primary operations surface) and on a phone (quick status updates).

## Assumptions that require real customer validation

Do not treat the following as facts in marketing copy, metrics, or implementation claims.

| ID | Assumption | Why it matters | How to validate |
|----|------------|----------------|-----------------|
| A1 | Target sellers currently track orders in WhatsApp + spreadsheets. | Determines import/entry UX. | Interview 5–10 sellers; ask to see their current sheet or chat flow. |
| A2 | COD is the dominant payment method for the first customers. | Profit model and status workflow. | Ask mix of prepaid vs COD by order volume. |
| A3 | Owners will type orders in by hand if the UI is fast. | Justifies no integrations in MVP. | Watch a trial of creating 10 orders; measure abandonment. |
| A4 | The cost fields above are the ones owners actually know. | Wrong fields make profit a lie. | Ask how they compute profit today; note missing fees (packaging, ads, returns shipping). |
| A5 | A single operator is the first user, not a team. | Defers roles and auth complexity. | Ask who else touches orders. |
| A6 | Courier integrations are wanted, but not required to switch from Excel. | Keeps MVP off third-party APIs. | Rank "must have now" vs "later". |
| A7 | Arabic-first UI is a buying reason, not just a nicety. | Prioritizes RTL quality. | Test English-only vs Arabic-first prototypes. |
| A8 | Owners think in daily/weekly cash, not fiscal months. | Default profit period. | Ask which date range they open first. |
| A9 | Phone number is the customer key they recognize. | Search and duplicate handling. | Observe how they find a customer in chat. |
| A10 | Willingness to pay for a focused tool vs free sheets. | Pricing is out of MVP build, but affects positioning. | Pricing conversations after a working prototype, not before. |

No customer quotes, conversion rates, or market-size numbers are included here because none have been collected in this repository.

## Success for the first implementation phase

The first phase succeeds if a seller can complete the primary use cases on a frontend with mock data, in Arabic on a phone and in English on a desktop, and if profit numbers match the documented formula for the sample orders.

It does not succeed by looking like a full SaaS, connecting to real couriers, or claiming traction.
