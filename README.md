# DocTruth — the README lie detector

Your docs say the app does X. DocTruth checks whether it actually does.

Give DocTruth a list of claims (plain-English sentences from your README) and
the app's URL. It turns each claim into a Kane CLI flow, runs it in a real
browser, and reports **TRUE** or **LIE** per claim — with a sealed evidence
pack (screenshots + logs) for every run.

When a claim fails, the result feeds back to the coding agent, which fixes
the app (or the doc) and re-runs Kane until the claim passes. That is the
closed loop: **agent builds → Kane verifies → Kane's verdict fixes the build**.

## The three bars (KaneCLI Hackathon)

1. **Works end to end** — `app/` is a real tiny store (PawShop). The flows
   run end to end in a real browser: browse a product, add to cart, check out.
2. **Kane caught something** — the checkout shipped with a real bug: the
   "Place order" button called a function that does not exist, so no order
   confirmation ever appeared. Kane's first run on claim C3 **failed** and
   proved the lie. Evidence: `evidence/run-001-fail/`.
3. **Closed loop** — the failure went back to the agent, which fixed the
   button handler, and Kane's re-run on the same claim **passed**.
   Evidence: `evidence/run-002-pass/`.

## Quickstart

```bash
# 1. Serve the demo app (any static server)
python3 -m http.server 8137 --directory app

# 2. Run the claims through Kane (needs `kane-cli login` first)
node tool/run.mjs --base-url http://localhost:8137

# 3. Read the verdict
cat report.md
```

## Layout

- `app/` — the demo store (PawShop): home + checkout.
- `claims.json` — the claims under test, each a plain-English sentence.
- `tool/run.mjs` — turns claims into `kane-cli run` flows, parses the
  NDJSON/JSON result, writes `report.md`.
- `evidence/` — sealed Kane evidence packs, one per run.
- `LOOP.md` — the loop, narrated: what Kane found, what changed, re-run.

## Requirements

- Node 20+
- [Kane CLI](https://www.testmuai.com/docs/kane-cli-quickstart)
  (`npm install -g @testmuai/kane-cli`, then `kane-cli login`)
- A TestMu AI account (10,000 free credits cover the demo many times over)
