# cs-churn-radar
AI-ready churn radar: scores a book of accounts on usage + contract signals and recommends retention plays, plugs into Google Antigravity and Claude.

An MCP server that scores Customer Success account health and surfaces churn risk
**before** the usage dashboard does. It reads a book of accounts, scores each one
on **usage/adoption** signals and **contract/renewal** signals, and hands an
agent the ranked portfolio, per-account breakdowns, recommended retention plays,
and a contract-aware save-motion timeline.

Because it speaks the **Model Context Protocol**, the same server plugs into both
**Google Antigravity** and **Claude** — configure once, use in either.

> Built as the working companion to *The Contract-Churn Playbook for CSMs*: the
> playbook explains what to look for; this turns it into a repeatable score and action.

---

## Why this exists

Most churn post-mortems blame the relationship or the price. But the signals were
usually visible months earlier — in flat usage, a quiet champion, an over-committed
spend floor, or an uncapped renewal uplift. CS Churn Radar makes those signals
machine-readable so a CSM (or an AI agent acting for one) can triage a large,
high-velocity book and act early instead of reacting at renewal.

## What it measures

**Usage / adoption health** (leading indicators, weight 55)
- Weekly active usage, 30-day usage trend, adoption depth, time-to-first-value,
  champion engagement, support sentiment.

**Contract / renewal risk** (weight 45)
- Consumption vs. commitment, renewal proximity vs. notice window,
  termination-for-convenience, uncapped price uplift, SLA misses & unclaimed
  credits, change-of-control events.

Each signal is normalized to a 0–1 risk fraction, weighted, and summed into a
**0–100 churn-risk score** with a **RED / YELLOW / GREEN** band.

## Tools exposed

| Tool | What it returns |
|------|-----------------|
| `cs_list_portfolio` | Ranked portfolio triage view (sort by risk / ARR / name, filter by band) |
| `cs_score_account` | Full signal-by-signal breakdown for one account |
| `cs_recommend_plays` | Retention plays for the signals firing on an account (contract levers first) |
| `cs_renewal_timeline` | Contract-aware save-motion timeline scaled to the notice window |
| `cs_portfolio_summary` | Book-level health: counts and ARR by band, ARR at risk, top risks |

## Quickstart

```bash
git clone <your-repo-url> cs-churn-radar
cd cs-churn-radar
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# run the test suite
python tests/test_scoring.py

# start the MCP server (stdio)
python server.py
```

Then connect it to Antigravity or Claude — see
[`examples/antigravity_and_claude_setup.md`](examples/antigravity_and_claude_setup.md).
Once connected, try:

> *"Summarize my portfolio health, then tell me which account to call first and why."*

> *"Score ACME-006 and give me the save plays — what do I do before discounting?"*

## Use your own data

The server reads `data/sample_accounts.json` by default. Point it at a real book
by setting `CHURN_RADAR_DATA` to your own JSON file (same schema). Field
definitions live in [`churn_radar/signals.py`](churn_radar/signals.py).

## Project layout

```
cs-churn-radar/
├── server.py                 # MCP server + tool definitions
├── churn_radar/
│   ├── signals.py            # signal definitions, weights, risk functions
│   ├── scoring.py            # score, band, plays, timeline
│   └── data.py               # data loading
├── data/sample_accounts.json # demo SMB portfolio (8 accounts)
├── examples/                 # Antigravity + Claude setup
└── tests/test_scoring.py     # unit tests
```

## Notes

The scoring weights and thresholds are deliberately simple, transparent, and easy
to tune — they are a starting framework, not a black box. Contract-risk logic is
an operational enablement aid for reading renewal risk; it is not legal advice.

MIT licensed.
