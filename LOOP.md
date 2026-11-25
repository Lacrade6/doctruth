# The Loop — what Kane found, what changed, re-run

This file narrates the closed loop for the DocTruth hackathon entry.
Every claim below links to its sealed Kane evidence pack.

## Infra note — the crash that blocked every run (24–25 Nov)

Every real run before 25 Nov 07:10 failed before the agent could act:
`Invalid port: ':1]'`. Root cause, proven 25 Nov: the Python runner
inside Kane (v16-runner) builds its API client with httpx, and httpx
chokes on the NO_PROXY entry `[::1]` present on this VM. Fix: run
kane-cli with NO_PROXY stripped of bracketed IPv6 entries
(`localhost,127.0.0.1,::1,198.19.0.1,198.19.0.2,fd8b:4f84:7d32:99::1,fd8b:4f84:7d32:99::2`).
Full diagnosis: `../DIAGNOSIS.md`. The first end-to-end run after the
fix was a toy probe (25 Nov 07:10): the agent opened PawShop, clicked
Cat Toy, reported the price $5 — passed in 43s (session
`d0d3b5c6-cf47-4177-89f0-dbd263915e8a`, 16.87 credits).

## Run 001 — infra failure, not a verification (24 Nov 22:29 WIB)

- Command: `node tool/run.mjs --base-url http://localhost:8137 --tag run-001`
- Result: **0 TRUE, 0 LIE, 3 ERROR** — every claim exited 2. Google Chrome
  was not installed on the VM (`Google Chrome is required but was not
  found at /usr/bin/google-chrome`), so Kane could not launch a browser.
  Evidence: `evidence/run-001/*.ndjson` (stderr shows the missing-Chrome
  message). Nothing was verified about the app.
- Fix: installed Google Chrome 158.0.8127.88 on the VM
  (`google-chrome-stable_current_amd64.deb`, kept in the workspace for
  fast reinstalls).

## Run 002 — infra failure, not a verification (24 Nov 22:32 WIB)

- Command: `node tool/run.mjs --base-url http://localhost:8137 --tag run-002`
- Result: **0 TRUE, 0 LIE, 3 ERROR** — every claim exited 2 with the CLI
  error `Invalid port: ':1]'` (no browser session started). Cause
  identified later (see infra note above): NO_PROXY `[::1]`.
  Evidence: `evidence/run-002/*.ndjson`.
- Then the VM was replaced: `/usr` installs (kane-cli + Chrome) were wiped.
  The workspace, the login profile (`~/.testmuai`), and the Chrome .deb
  survived.

## Run 003 — INVALID, not a verification (24 Nov ~23:40 WIB)

- Command: `node tool/run.mjs --base-url http://localhost:8137 --tag run-003`
- Result on paper: **0 TRUE, 3 LIE**. Discarded: all three LIEs are the
  same stall — `AP produced no action for 3 consecutive steps`. The agent
  never acted, so nothing about the app was verified. The stall came from
  the background session the run was launched in (its fetches to the
  TestMu API failed), not from the app. Evidence: `evidence/run-003/*.ndjson`.
  Do not treat run-003 as evidence for or against any claim.

## Run 004 — the first real verification (25 Nov ~07:19 WIB)

- Command: `node tool/run.mjs --base-url http://localhost:8137 --tag run-004`
- Result: **2 TRUE, 1 LIE, 0 ERROR**.
- C1 TRUE — clicked the "Cat Toy" product name; the detail view opened
  showing "Cat Toy" with "Price: $5". Session
  `e6645e45-24c5-4846-9edc-60cccf7febe6`.
- C2 TRUE — clicked "Add to cart"; the header updated to "Cart: 1 items —
  Checkout". Session `7a4965fc-e847-4ed2-b71b-2dcc2133dfd2`.
- **C3 LIE — the catch.** Kane reported: "The checkout button tries to use
  a submission action that is missing, so the order cannot be completed.
  Restore the checkout submission action and verify that clicking Place
  order displays a confirmation with an order number." Console evidence:
  `submitOrder is not defined`. Session
  `4631fcd2-bb59-4542-99f1-5a6f668185c1`.
- Evidence: `evidence/run-004/*.ndjson`, `report.md` (run-004).

## The fix (driven by Kane's C3 report)

- What Kane reported: the checkout button calls a submission action that
  does not exist (`submitOrder is not defined`).
- Change: one line in `app/checkout.html` — the "Place order" button now
  calls the real `placeOrder()` function (`onclick="placeOrder()"` instead
  of `onclick="submitOrder()"`), which renders the confirmation with a
  generated order number (`PS-######`). The comment above the button
  records that Kane caught the bug in run-004.
- No other behavior changes. The bug was the only thing standing between
  the claim and the app.

## Run 005 — the re-run: loop CLOSED (25 Nov ~07:27 WIB)

- Command: `node tool/run.mjs --base-url http://localhost:8137 --tag run-005`
- Result: **3 TRUE, 0 LIE, 0 ERROR**.
- C1 TRUE, C2 TRUE — same as run-004.
- **C3 TRUE — the fix confirmed.** Kane filled in the name and address,
  clicked "Place order", and the checkout page displayed an order
  confirmation with an order number. Session
  `dd7c22c0-fb4d-4e3c-a046-d153a662aeff`.
- Evidence: `evidence/run-005/*.ndjson`, `report.md` (run-005).
- The loop is closed: agent built, Kane verified, Kane's verdict fixed the
  build, Kane confirmed the fix.

## Why this is the loop, not a scripted pass

- The bug is real and present in the shipped app at run 004; the failure
  is observed, not staged in the test.
- The fix is driven by Kane's failure detail, not by a pre-written patch:
  the agent reads what Kane saw (`submitOrder is not defined`) and changes
  the app accordingly.
- Both runs are sealed evidence packs (screenshots + console/network logs
  per step) under `evidence/`, reproducible with the commands above.
