# Campfire Analyst

Your portfolio is a fingerprint. Campfire Analyst reads it.

A Claude skill for Zerodha users. Connects to your account, runs a full portfolio analysis, and produces three files.


---

## Output

| File | What it contains |
|------|-----------------|
| `campfire-dashboard.html` | Four-tab HTML app — Summary (headline + quick wins), Fix List (task center), Analysis (Vitals + portrait + sectors), X-Ray (look-through inside your mutual funds) |
| `investment.md` | Full wealth document: allocations, sector breakdown, consolidated views, gaps |
| `taste.md` | Lifestyle profile generated through a prediction game — phone, car, hotel tier inferred from your holdings |

---

## How It Works

**Campfire Vitals**
Scores your portfolio across 8 health metrics (0–100): diversification, cost efficiency, risk calibration, growth momentum, rebalancing discipline, income streams, regime adaptability, capital deployment. Each score has a one-line interpretation and a concrete fix if it's low.

**Fix List**
Every structural issue in your portfolio — fragmented holdings, underweight asset classes, dead positions, idle cash — listed with impact rating, effort estimate, and specific next steps. Lives inside the dashboard as the second tab.

**Prediction Game**
Claude tries to predict your lifestyle from your holdings alone — one item at a time, with the reasoning shown. You confirm or correct. Generates `taste.md` at the end.

**Fragmentation Detection**
Many Indian investors accumulate the same asset across multiple vehicles without noticing — 5 Nifty funds, 4 gold ETFs. Campfire Analyst flags this and tells you what to consolidate.

**Dual Portfolio Merging**
Direct equity and mutual funds are analysed together, not separately. The real allocation only shows up when you merge both views.

---

## Installation

1. Download `campfire-analyst.zip` from the [Releases](../../releases) page
2. Unzip it
3. Open Claude → **Customize** → **Add Skill** → upload the folder
4. Type `/campfire-analyst` to run

**Requires:** Zerodha account with Kite MCP connected in Claude.

---

## Usage

```
/campfire-analyst
```

You'll get a Zerodha login link. Authorize it and the analysis runs. No configuration needed.

To run a specific part:
```
"Just give me my Campfire Vitals report"
"Generate my investment.md"
"Let's do the prediction game"
"Show me my fix list"
```

No Zerodha? Paste your holdings directly — stock name, quantity, average buy price. The skill works with whatever you give it.

---

## What It Doesn't Do

Campfire Analyst only reads your holdings data. It does not place orders, execute trades, or modify your account in any way.

---

## File Structure

```
campfire-analyst/
├── SKILL.md                      ← skill instructions
├── LICENSE.txt                   ← Apache 2.0
└── references/
    ├── vitals-metrics.md         ← scoring rubrics for all 8 Campfire Vitals
    ├── dashboard-template.md     ← HTML dashboard spec (four tabs)
    ├── task-center-template.md   ← Fix List tab spec
    ├── xray-template.md          ← Portfolio X-Ray tab spec
    ├── archetypes.md             ← investor personality archetypes
    └── prediction-chains.md      ← prediction game methodology
```

---

## License

Apache 2.0 — see [LICENSE.txt](LICENSE.txt)

Copyright 2026 Subodh Kolhe
