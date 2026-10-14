# ADR 0006 — Egyptian Arabic dialect for all customer copy

- **Status:** Accepted
- **Date:** 2026-10-14

## Context

The customers are Egyptian and order in colloquial Egyptian Arabic, not Modern
Standard Arabic (فصحى). Stiff MSA copy reads as foreign and hurts conversion; the
restaurant's own name and menu categories are colloquial.

## Decision

All **customer-facing copy is written in Egyptian Arabic dialect**.

- This covers the Bot's replies, menu labels, checkout confirmations, the
  `جاري التحضير` message, and the presentation deliverable.
- **Technical terms, code, and identifiers stay in Latin script** (e.g. `Order`,
  `placed`, `get_menu`, `order_lines`).
- The Agent is prompted to mirror the customer's dialect.
- Layout is **RTL** with an Arabic web font (Cairo).

## Consequences

- **Good:** the Bot sounds like the restaurant, not a form letter.
- **Good:** clean separation — humans read Arabic, the system speaks Latin.
- **Cost:** copy needs an Arabic-speaking reviewer; machine translation to MSA
  would read wrong and must be avoided.
