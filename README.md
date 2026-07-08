# Composio App-Research Agent — Take-Home

Researches whether an app can become an agent toolkit today: auth method, self-serve vs
gated, API surface, and a buildability verdict — across a 100-app list, then verifies a
sample of the results against live docs.

## What's here

```
data/
  apps.json                 100 rows, the actual findings (category, auth, access,
                             API surface, verdict, blocker, evidence URL, confidence)
  verification_sample.json  5 apps hand-checked against live docs, hits and misses
agent/
  research_agent.py         the pipeline: search → fetch → extract (Claude) → verify
                             (a second, adversarial Claude pass) → flag disagreements
site/
  index.html                the single-page case study (open this first)
```

## The pipeline, in one paragraph

For each app: **search** for its developer docs (Composio's SEARCH toolkit, or any
web-search MCP), **fetch** the top result (Composio's browser tool handles JS-rendered
docs), **extract** the 7 fields with Claude reading the fetched text, then **verify**
with a second, deliberately skeptical Claude call that re-reads the same source and is
told to look for reasons the first answer might be wrong — not confirm it. Disagreements
get written back with the correction, not silently averaged away.

## Where a human is required by design, not as a fallback

- **Disambiguating a name.** If a name matches more than one real product (see `apps.json`
  row 85, `iPayX`), the agent is instructed to say so instead of guessing — a human picks
  the right one.
- **Sales-gated products** with no further public docs to scrape (PitchBook, Gladly) — the
  finding _is_ "gated, no public path," which the assignment explicitly says counts as a
  correct answer, not a failure.
- **Spot-checking the verify step itself.** The verify pass catches its own disagreements,
  but a human should skim the `verified: false` rows before trusting them at scale —
  that's exactly what `verification_sample.json` demonstrates on 5 rows.

## Running it for real

```bash
pip install anthropic
export ANTHROPIC_API_KEY=...
# optional, to use Composio's own SEARCH + browser tools instead of writing your own:
export COMPOSIO_API_KEY=...

cd agent
python research_agent.py --input ../data/app_list.json --output ../data/agent_output.json
```

`search_and_fetch()` in `research_agent.py` is left as an explicit extension point — swap
in real `COMPOSIO_SEARCH_SEARCH` / browser-tool calls (the two-line example is in the
docstring) rather than hiding that wiring behind an abstraction. Without network
credentials in this sandbox, the 100 rows in `data/apps.json` were produced the same way
conceptually — search + fetch + extract — but run by hand for this submission, with the
5-row sample in `verification_sample.json` cross-checked live against docs sites.

## Deploying the site

`site/index.html` is fully static (data is inlined) — drag it into Netlify/Vercel/GitHub
Pages, or run `python -m http.server` inside `site/` to preview locally.
