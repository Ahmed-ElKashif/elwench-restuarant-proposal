# ADR 0004 — Delivery fee lives outside the system

- **Status:** Accepted
- **Date:** 2026-10-14

## Context

The delivery fee is set by the delivery person and varies by area/distance. The
restaurant does not want it stored, shown, or entered anywhere in the system, and
does not want to expose per-area pricing.

## Decision

The **Delivery fee is entirely out-of-system**.

- It is **never stored**, **never shown** in the dashboard, and **never entered**
  by staff.
- There is **no optional logging** field for it.
- The Bot's only reference is a fixed line: *"plus delivery fee, paid cash on
  delivery."*
- **Item total** — the sum of the Cart's Lines — is the only amount the system
  computes or displays.

## Consequences

- **Good:** no pricing model, no area table, no reconciliation; matches how the
  business actually operates.
- **Good:** the Order record and totals stay unambiguous about what is and isn't
  included.
- **Cost:** the customer cannot be told a final payable number online. This is
  intentional — the system is **COD** and the courier collects the fee.
