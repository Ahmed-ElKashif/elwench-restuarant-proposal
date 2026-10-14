<div align="center">

<img src="logo.jpg" alt="El Wench — الونش" width="132" height="132" />

<h1>El Wench — WhatsApp Ordering Assistant</h1>

<p dir="rtl"><b>مساعد أوردرات ذكي على واتساب — مطعم الونش</b></p>

<p>An Egyptian-Arabic AI agent that takes orders end to end, hands them to the kitchen
dashboard, and collects feedback — running entirely on the restaurant's own PC.</p>

<img alt="status" src="https://img.shields.io/badge/status-presentation-B01414?style=for-the-badge&labelColor=1F1414" />
<img alt="payment" src="https://img.shields.io/badge/payment-cash%20on%20delivery-F0B050?style=for-the-badge&labelColor=1F1414" />
<img alt="dialect" src="https://img.shields.io/badge/Arabic-Egyptian%20dialect-B01414?style=for-the-badge&labelColor=1F1414" />

<br>

<img alt="React" src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
<img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" />
<img alt="Node.js" src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" />
<img alt="Express" src="https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white" />
<img alt="MongoDB" src="https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white" />
<img alt="Redux Toolkit" src="https://img.shields.io/badge/Redux%20Toolkit-764ABC?style=for-the-badge&logo=redux&logoColor=white" />
<img alt="Tailwind CSS" src="https://img.shields.io/badge/Tailwind%20CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" />
<img alt="Chart.js" src="https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white" />
<img alt="ngrok" src="https://img.shields.io/badge/ngrok-1F1E37?style=for-the-badge&logo=ngrok&logoColor=white" />
<img alt="OpenRouter" src="https://img.shields.io/badge/OpenRouter-6467F2?style=for-the-badge&logo=openai&logoColor=white" />
<img alt="WhatsApp Cloud API" src="https://img.shields.io/badge/WhatsApp%20Cloud%20API-25D366?style=for-the-badge&logo=whatsapp&logoColor=white" />

</div>

---

> This repository contains a **presentation**, not the running system. There is
> no backend, database, or build step here.

## What this is

A single, self-contained HTML document (`index.html`) that walks the client
through the whole proposal — the idea, the order flow, the architecture, the AI
layer, safety, checkout, feedback, the admin dashboard, reports, running cost,
and a live interactive demo. It is a static page: open it in a browser and it
works.

## Files

| File | Purpose |
|---|---|
| `index.html` | The presentation itself (single file, self-contained). |
| `logo.jpg` | Brand logo used in the header and demo. Must sit next to `index.html`. |
| `menu.js` | The restaurant's menu source (categories, items, sizes, upsell map). Reference data for the bot. |
| `CONTEXT.md` | Domain glossary — the shared vocabulary (Order, Cart, Variant, Handoff, ETA, ...). |
| `docs/adr/` | Architecture Decision Records `0001`–`0006`. |

## Viewing it

- **Quickest:** double-click `index.html` (or drag it into a browser).
- **Local server (optional):**
  ```bash
  python -m http.server 8000
  # then open http://localhost:8000/
  ```
- An internet connection is required on first load for the CDN assets
  (Tailwind CSS, Chart.js, and the Cairo font). The page is designed for
  desktop and every mobile viewport (no page-level horizontal scrolling).

## Tech

- **This presentation:** static **HTML** with **Tailwind CSS** (CDN) and
  **Chart.js** (CDN).
- **Proposed product stack:** React + **TypeScript** (state via **Redux Toolkit**)
  frontend; **Node.js + TypeScript** (Express) API; **MongoDB**; **OpenRouter**
  (`qwen/qwen3.8-27b`); **WhatsApp Cloud API**; exposed via **ngrok**.
- **Google Fonts — Cairo**, RTL layout, Egyptian Arabic copy.
- Nav scroll-spy (IntersectionObserver) and a scripted **live demo** in vanilla
  JavaScript (no framework).

## Presentation sections

1. **الفكرة** — the idea: the four everyday wins.
2. **طريقة الطلب** — the order journey, step by step (Connected chain).
3. **مراحل الطلب** — the five order stages.
4. **البنية** — architecture: the assistant decides, the code executes, all on the restaurant's PC.
5. **المساعد الذكي** — the AI layer and the tools it may call.
6. **الأمان والأدب** — abuse handling and guardrails.
7. **الخروج والدفع** — checkout and cash-on-delivery.
8. **التقييم** — post-delivery rating and feedback.
9. **لوحة الإدارة** — the staff dashboard.
10. **التقارير** — read-only reports and charts.
11. **التكلفة** — expected monthly cost.
12. **الدخول والحسابات** — staff login and roles.
13. **تجربة حيّة** — live interactive demo (chat + dashboard).

## Architecture at a glance

- **Hybrid agent.** On every turn the LLM (`qwen/qwen3.8-27b` via OpenRouter)
  decides freely whether to reply in plain text or to call a tool. It is **not**
  forced through a tool on every message.
- **Code writes state.** Regardless of the agent's choice, only deterministic
  code validates inputs, computes prices/totals/ETA, and writes data. The model
  never computes a number on its own.
- **No Meta catalog.** The menu and prices live in code (`menu.js`); the agent
  calls our own functions as tools. Price/item edits from the dashboard apply
  instantly with no Meta sync.
- **On-prem.** Node + TypeScript + Express + MongoDB run on the restaurant's own
  PC, with a React + TypeScript (Redux Toolkit) dashboard, exposed over an ngrok
  tunnel — no monthly server bill.
- **Delivery fee** is intentionally outside the system: never stored, shown, or
  entered. The bot only says "plus delivery fee, paid cash on delivery."

## Monthly operating cost (estimate)

Two variable lines scale with order volume; everything else is fixed at zero.
Model: `qwen/qwen3.8-27b` (OpenRouter, $0.40 / 1M input, $3.00 / 1M output).
Meta: first 1,000 service messages per number per month free, then ~$0.0036 each;
inbound free; per-message billing effective Oct 1, 2026. Assumes 1 USD ≈ 50 EGP.

| Orders / month | Assistant (OpenRouter) | WhatsApp messages (Meta) | Approx. total |
|---|---|---|---|
| 1,000 | 750–1,250 EGP | ~1,640 EGP | 2,400–2,900 EGP |
| 2,500 | 1,900–3,100 EGP | ~4,400 EGP | 6,300–7,500 EGP |
| 5,000 | 3,750–6,250 EGP | ~9,000 EGP | 12,750–15,250 EGP |

## Domain & decisions

- **Domain glossary:** [`CONTEXT.md`](CONTEXT.md).
- **Architecture Decision Records:**
  - [ADR 0001 — Hybrid Agent, not full LLM](docs/adr/0001-hybrid-agent-not-full-llm.md)
  - [ADR 0002 — Express + MongoDB (over Fastify + Postgres)](docs/adr/0002-express-mongodb.md)
  - [ADR 0003 — On-prem PC behind an ngrok free tunnel](docs/adr/0003-on-prem-pc-ngrok-free.md)
  - [ADR 0004 — Delivery fee lives outside the system](docs/adr/0004-delivery-fee-out-of-system.md)
  - [ADR 0005 — Three-step abuse ladder with temporary suspension](docs/adr/0005-abuse-suspend-policy.md)
  - [ADR 0006 — Egyptian Arabic dialect for all customer copy](docs/adr/0006-egyptian-dialect-copy.md)
