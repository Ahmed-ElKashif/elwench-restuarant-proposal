# ADR 0001 — Hybrid Agent, not full LLM

- **Status:** Accepted
- **Date:** 2026-10-14

## Context

The Bot must converse naturally in Egyptian Arabic, browse a large menu (16
categories, ~130 items, several with size Variants), build a Cart, and place an
Order. Two naive designs are on the table:

1. **Full LLM** — let the model hold the menu, prices, cart, and totals in its
   context and free-form the whole flow.
2. **Full deterministic** — buttons and canned lists only, no natural language.

A full LLM cannot be trusted with money-adjacent arithmetic, availability, or a
~130-item catalog inside a chat window: it will hallucinate prices, forget Lines,
and produce totals that don't match the Menu. A purely deterministic UI cannot
handle the fuzzy, colloquial way customers actually order ("عايز حاجة سخنة",
"نص كيلو فول").

## Decision

Adopt a **hybrid Agent**: the LLM (`qwen/qwen3.8-27b` via OpenRouter) converses
with the customer and **decides for itself** whether to call a Tool or not;
deterministic code owns all state and maths.

- **Tool use is model-initiated and optional.** On every turn the Agent makes an
  *Agent decision*: answer the customer in plain text, **or** call a Tool to act.
  It is **not** forced through a Tool, and there is no fixed router — ordinary
  chit-chat ("ازيك؟", "بتفتحوا امتى؟", "شكراً") is answered directly with no Tool
  call at all.
- When the customer wants an **order action**, the Agent calls the matching Tool
  (`add_to_cart`, `remove_item`, `place_order`, …).
- The catalog is **never** in the prompt in full — the Agent calls `get_menu` /
  menu-search Tools when it needs to look something up.
- Price, total, availability, and variant resolution live in code. **Every Tool
  call is validated before it takes effect.** Tool calls use the model's native
  tool-calling / structured-output support.
- The Agent may produce the Arabic phrasing; it may **not** invent a price, a
  total, or an item that does not exist. Whatever the Agent decides, **code is the
  only writer of state.**

This generalises the v2 spec's `simulatedLLMSuggester` idea: the LLM suggests and
phrases, code decides.

## Consequences

- **Good:** totals are always correct; the Menu is the single source of truth;
  abuse and availability rules can be enforced without trusting the model.
- **Good:** swapping models is low-risk — the Tool contract is the seam.
- **Cost:** we maintain a Tool surface and validators.
- **Constraint:** the model must support tool calling; the chosen model does.
  `reasoning_effort` stays low / thinking off to keep latency and cost down.
