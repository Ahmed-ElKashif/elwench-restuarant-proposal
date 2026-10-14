# El Wench — Domain Context

Glossary for the **El Wench (الونش)** WhatsApp ordering bot. Terms are the shared
vocabulary used across the spec doc, ADRs, and (future) code. Definitions are
normative: when prose and this file disagree, this file wins.

- **El Wench (الونش)** — the restaurant. A single-branch Egyptian fast-food
  business (فول، فلافل، بيتزا، كريب، حواوشي، مقرمشات). The system serves exactly
  one restaurant; no multi-tenant support.
- **Bot** — the WhatsApp-facing AI assistant. Customer-facing name is Arabic;
  internally it is the *Agent* plus deterministic code.
- **Customer** — the person messaging the restaurant's WhatsApp number to place
  an order. Identified by phone number. Not authenticated in any way.
- **Agent** — the LLM (`qwen/qwen3.8-27b` via OpenRouter) that converses with the
  customer in Egyptian Arabic. On **every turn it decides freely** whether to reply
  in plain text or to call a *Tool* — it is **not** forced through a Tool. The
  Agent never computes prices, totals, or availability itself.
- **Tool** — a typed function the Agent is allowed to call (e.g. `get_menu`,
  `add_to_cart`, `place_order`). Tool use is **model-initiated and optional**: for
  ordinary chat the Agent answers directly with no Tool call; for an order action
  it calls the matching Tool. Every Tool call is validated by deterministic code
  before it takes effect. See `docs/adr/0001-hybrid-agent-not-full-llm.md`.
- **Agent decision** — the Agent's per-turn choice between a **direct reply** (no
  Tool) and a **Tool call**. It is the Agent's own call, not a fixed router; the
  operator never sees or controls it. Regardless of the choice, code remains the
  only writer of state.
- **Handoff** — the transition from Bot to a human (Manager/Owner) that happens
  when the Agent cannot serve a request, or on abuse escalation. The Bot stops
  replying for that conversation.

## Catalog & cart

- **Category** (`فئة`) — a top-level menu grouping, e.g. `فول`, `بيتزا`,
  `كريب فراخ`. 16 categories in the seeded menu.
- **Item** (`صنف`) — an orderable product, e.g. `فول مدمس`, `بيتزا مارجريتا`.
- **Variant** (حجم / `size`) — a size-specific form of an Item carrying its own
  price, e.g. بيتزا: `صغير` / `وسط` / `كبير`. Items without variants have a
  single price. A Line always resolves to exactly one priced Variant.
- **Menu** — the full set of Categories and Items, seeded from the restaurant's
  source file (`menu.js`) into the `menu` collection. Editable only by the Owner.
- **Recommendation (`RECS`)** — an item-to-item suggestion map used by the Bot to
  upsell (e.g. suggesting a side to a main). Seeded from the same source file.
- **Cart** — the ephemeral, in-conversation list of Lines for a customer. Lives
  in server memory, keyed by phone number. Never persisted.
- **Line** (`سطر طلب`) — one Item+Variant with a quantity, inside a Cart or an
  Order.
- **Item total** — the sum of a Cart's Lines. **Delivery fee is not part of it.**

## Ordering

- **Order** (`طلب`) — a Customer's confirmed request with Lines, contact number,
  address, order type, and a computed ETA. The only durable record of a
  transaction.
- **Order type** — `delivery` (توصيل) or `pickup` (استلام من الفرع).
- **COD (Cash on Delivery)** — the only payment method. The Bot may charge the
  customer **nothing** online; money is collected in cash on delivery/pickup.
- **Delivery fee** — the amount the delivery person collects, set entirely outside
  the system. It is **never stored, shown, or entered** in the dashboard. The Bot
  only ever says *"plus delivery fee, paid cash on delivery."* See
  `docs/adr/0004-delivery-fee-out-of-system.md`.
- **Checkout** — the step where the Bot asks for and re-confirms the contact
  number and address **every time**, then restates item total + COD + ETA before
  placing the Order. No saved addresses, no reorder memory.
- **Customer confirm** — the Customer's explicit "yes" that triggers `place_order`.
- **Manager confirm** — the Manager's explicit acceptance in the dashboard that
  starts fulfilment. Distinct from *Customer confirm*; both are required.
- **ETA** — the estimated time to delivery/collection. **Set dynamically by the
  Manager in the dashboard** (defaults: delivery 60 min, pickup 30). The
  notification message carries the current value with no LLM involvement.

## Order lifecycle

The Order state machine is deliberately small:

1. **placed** — Customer confirmed; internal notification sent to the dashboard.
2. **confirmed** — Manager accepted the order in the dashboard.
3. **preparing** (`جاري التحضير`) — the single message the Customer receives. Sent
   by deterministic code, **not** the Agent.
4. **delivered** — Manager marked it done. Starts the Feedback timer.

There is no `out-for-delivery`, `cancelled`, or `on-the-way` status. Delivery is
tracked by the human courier outside the system.

## Safety & abuse

- **Abuse** — a customer interaction that isn't a legitimate order attempt:
  harassment, profanity, spam, or repeated nonsense. Classified in code, not by
  the Agent alone.
- **Loop** — the customer repeating the same conversational state 3 times without
  progress. A Loop is treated as Abuse.
- **Abuse ladder** — the 3-step response:
  1. friendly redirect,
  2. notify Manager + 15-minute **Suspension**,
  3. notify Owner + Handoff.
- **Suspension** — a temporary (15-minute) block on the Bot replying to one phone
  number. Ephemeral; not persisted.
- **Bad-word list** — an admin-editable list of terms used to flag **Feedback**
  comments. A flag notifies both Manager and Owner.

## Modes & availability

- **Master switch** (`accepting orders`) — an Owner/Manager dashboard toggle that,
  when off, pauses all new orders.
- **Traffic-jam mode** — a dashboard toggle with an editable delay (default 30
  min) added to every ETA, plus a delay note appended to messages. Not a hard
  pause.
- **Off-hours message** — an editable static message + on/off switch + schedule,
  shown outside business hours. Separate from Traffic-jam mode.

## Feedback

- **Feedback** — a request for a rating sent automatically **2 hours** after an
  Order is `delivered` (configurable 2–3h). Only one per Order.
- **Rating** — an integer 1–5, plus an optional comment. Stored for **Analytics**.

## People

- **Owner** — the Super Admin. The only person who can create/remove Managers,
  edit the Menu, edit Settings, and reset forgotten passwords. Receives the final
  abuse escalation.
- **Manager** — dashboard staff. Answers orders, sets ETA, toggles modes, marks
  orders delivered, and views Analytics. Cannot manage staff or the Menu.
- Two roles only. No MFA, no invitations/resend flow. Login is username +
  password with an in-memory session.

## Data & reporting

- **Analytics / Reports** — read-only dashboards (KPIs + Chart.js line/bar/donut,
  day/week/month/year). **No CSV export.**
- **Persisted entities** — `orders`, `order_lines` (embedded), `feedback`,
  `staff_users`, `menu`, `settings`.
- **Ephemeral state** — the Customer conversation/session (server memory keyed by
  phone). Resets at Checkout and after Feedback; never written to the database.

## Glossary note

Reasoning-visible terms in the spec that are **not** domain concepts: "hybrid
agent", "atomic claim", "ngrok tunnel" — see the ADRs for those design decisions.
