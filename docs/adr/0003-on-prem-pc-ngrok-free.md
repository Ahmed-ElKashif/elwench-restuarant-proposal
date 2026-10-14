# ADR 0003 — On-prem PC behind an ngrok free tunnel

- **Status:** Accepted
- **Date:** 2026-10-14
- **Risk:** flagged, accepted by owner

## Context

The backend must be reachable by WhatsApp/Meta webhooks. Options were a cloud
host (recurring cost + ops) or the restaurant's own always-on PC exposed via a
tunnel. The client has a decent PC and a small order volume.

## Decision

Run the backend on the **client's PC** and expose it with **ngrok free tier**.

## Consequences

- **Good:** zero hosting spend; the owner controls the machine; quick to stand up.
- **Risk — monthly caps:** ngrok free allows ~20k requests **and** ~1 GB traffic
  per month. Media/photo traffic counts, so the Bot should stay text-first.
- **Risk — browser interstitial:** the free tier shows a warning page for normal
  browser hits (not Meta webhooks). Acceptable because the only public consumer is
  the webhook.
- **Risk — flaky home connection / PC off** = Bot offline. The dashboard should
  show a clear online/offline indicator, and the owner must keep the PC on.
- **Mitigation:** stable dev domain on the free tier so the webhook URL is fixed.
