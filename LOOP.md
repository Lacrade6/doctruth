# The Loop — what Kane found, what changed, re-run

This file narrates the closed loop for the DocTruth hackathon entry.
Every claim below links to its sealed Kane evidence pack.

## Run 001 — the first verification (to be filled after the run)

- Command: `node tool/run.mjs --base-url <app-url> --tag run-001`
- Result: _pending — awaiting `kane-cli login`._
- Expected: C1 TRUE (product detail opens), C2 TRUE (cart count goes to 1),
  **C3 LIE** — the "Place order" button calls `submitOrder()`, a function
  that does not exist, so no order confirmation ever appears. The README
  claim "clicking Place order shows an order confirmation containing an
  order number" is false in the live app. Kane should catch this.

## The fix (to be filled after run 001)

- What Kane reported: _pending._
- Change: wire the "Place order" button to the real `placeOrder()` function
  (one-line fix in `app/checkout.html`), which renders the confirmation
  with a generated order number.
- No other behavior changes. The bug was the only thing standing between
  the claim and the app.

## Run 002 — the re-run (to be filled after the fix)

- Command: `node tool/run.mjs --base-url <app-url> --tag run-002`
- Expected: **C1, C2, C3 all TRUE.** The loop closes: agent built, Kane
  verified, Kane's verdict fixed the build, Kane confirmed the fix.

## Why this is the loop, not a scripted pass

- The bug is real and present in the shipped app at run 001; the failure
  is observed, not staged in the test.
- The fix is driven by Kane's failure detail, not by a pre-written patch:
  the agent reads what Kane saw and changes the app accordingly.
- Both runs are sealed evidence packs (screenshots + console/network logs
  per step) under `evidence/`, reproducible with the commands above.
