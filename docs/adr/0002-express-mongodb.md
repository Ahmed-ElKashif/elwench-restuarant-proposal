# ADR 0002 — Express + MongoDB (over Fastify + Postgres)

- **Status:** Accepted
- **Date:** 2026-10-14

## Context

v2 assumed **Fastify + PostgreSQL**. The data is a single restaurant's Orders,
embedded Lines, Feedback, staff users, a seeded Menu, and Settings. There is no
multi-tenant requirement, no heavy analytics query load, and no team of engineers
to operate a schema-migration pipeline. The Menu is naturally a document
(nested Categories → Items → Variants).

## Decision

Use **Node + Express + MongoDB (Mongoose)**.

- Orders embed their `order_lines`.
- The Menu and Recommendations are seeded from the restaurant's source file
  (`menu.js`) into the `menu` collection.
- Order state transitions that must not race (two Managers accepting the same
  Order) use an **atomic** `findOneAndUpdate` claim, not read-then-write.

## Consequences

- **Good:** the document model matches the Menu and Order shape; no migrations to
  babysit; Express is familiar and easy for the client to host.
- **Good:** flexible Menu editing (Owner-only) is straightforward.
- **Cost:** weaker relational guarantees — acceptable given the tiny schema and
  single writer.
- **Note:** the v2 Postgres DDL, migration talk, and Fastify-vs-NestJS comparison
  are dropped from the deliverable.
