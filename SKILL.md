---
name: campfire-analyst
description: "Your portfolio is a fingerprint. Campfire Analyst reads it. Connects to Zerodha, runs a full-stack analysis — sector breakdown, Campfire Vitals scoring, fragmentation audit, actionable task list, and a Portfolio X-Ray that looks inside every mutual fund to reveal your true underlying exposure — and then plays a prediction game: your phone, your car, your hotel tier, all inferred from your holdings. Outputs: campfire-dashboard.html (four-tab visual app), investment.md, taste.md. Trigger on: portfolio review, Campfire Vitals report, investment.md, taste.md, predict me from stocks, campfire analyst, task center, consumer identity, financial health check, portfolio x-ray, look-through analysis."
author: Subodh Kolhe
license: Apache 2.0 — complete terms in LICENSE.txt
---

# Campfire Analyst

---

## For Users: How to Use This Skill

### What it does
Campfire Analyst reads your Zerodha portfolio and produces a complete picture of your investments — what you own, how it's performing, what's broken, and what your money reveals about you as a person.

It generates three files:
- **campfire-dashboard.html** — four-tab visual app: Summary (headline + quick wins), Fix List (task center), Analysis (Vitals + portrait + sectors), and X-Ray (look-through inside your mutual funds)
- **investment.md** — a detailed wealth document you can save or share with an advisor
- **taste.md** — a lifestyle identity profile built through a prediction game

### How to start
Just type `/campfire-analyst` in the chat. That's it. No setup, no options to configure.

You'll be asked to log into your Zerodha account via a one-click link. Once authorized, the analysis runs automatically.

### What happens next
1. You'll see the **Campfire Dashboard** — a four-tab HTML file. The **Summary** tab leads with a single headline finding and the three most actionable quick wins. The **Fix List** tab has the full portfolio task center. The **Analysis** tab shows Campfire Vitals, style portrait, thematic map, concentration risk (combined across direct equity and MFs), and benchmark comparison. The **X-Ray** tab shows the full breakdown of what's inside each mutual fund — the evidence behind the concentration numbers, loaded automatically on page open.
2. **investment.md** follows immediately — the full wealth document with deep allocation tables.
3. Then the **prediction game** begins — Claude tries to guess your phone, car, watch, hotel preference and more from your portfolio alone, one prediction at a time. At the end it generates taste.md.

### If you only want part of it
You can ask for specific pieces directly:
- *"Just give me my Campfire Vitals report"*
- *"Generate my investment.md"*
- *"Let's do the prediction game"*
- *"Show me my fix list"*
- *"Run Portfolio X-Ray"*

### If you don't use Zerodha
You can still run the analysis manually. Share your holdings as a screenshot, CSV, or just paste your top 20-30 positions with quantities and average buy prices. Claude will work with whatever you provide.

### What it doesn't do
It doesn't execute any trades, place orders, or modify your account in any way. It only reads your holdings data.

---

## For Skill Authors: Framework Reference

A seven-part portfolio intelligence framework that transforms raw holdings data into a complete financial + consumer identity profile.

When triggered, this skill runs **seven sequential steps** on the user's portfolio:

| # | Analysis | What It Produces | Reference File |
|---|----------|-----------------|----------------|
| 1 | Deep Portfolio Analysis | Sector breakdown, winners/losers, asset class mix, MF breakdown, thematic map, expense audit, liquidity, yield, beta | (computed internally — feeds dashboard) |
| 2 | Campfire Vitals Report | 8 custom financial health markers scored 0-100 | `references/vitals-metrics.md` (computed internally — feeds dashboard) |
| 3 | Investment Style Portrait | Written paragraph characterising the investor's style and blind spots | (computed internally — feeds dashboard) |
| 4 | Portfolio X-Ray Classification | Fund tier classification + JavaScript engine for browser-side look-through | `references/xray-template.md` (embedded in dashboard HTML) |
| 5 | Campfire Dashboard | Four-tab HTML file — **Summary** · **Fix List** · **Analysis** · **X-Ray** | `references/dashboard-template.md`, `references/task-center-template.md`, `references/xray-template.md` |
| 6 | investment.md Generation | Complete wealth document with deeper consolidated allocation tables | (file output) |
| 7 | taste.md Consumer Identity | Archetype extraction → lifestyle prediction game | `references/archetypes.md`, `references/prediction-chains.md` |

**CRITICAL: Every analysis must be derived fresh from THIS user's data. Do not carry over any assumptions, archetypes, predictions, or personal details from any prior conversation. Every user is unique. Every portfolio tells a different story.**

---

## Step 0: Pull Data

### Primary: Kite MCP

```
kite:get_holdings        → All stock/ETF positions
kite:get_mf_holdings     → All mutual fund positions
kite:get_margins         → Cash balance
kite:get_positions       → Intraday/F&O positions (optional)
```

If `kite:get_mf_holdings` fails or returns empty, note this and proceed with direct equity only. Flag the gap.

### Fallback: Manual Upload
If Kite MCP is not connected, ask for:
- Holdings screenshot or CSV export
- Mutual fund statement (CAS PDF or broker export)
- Or at minimum: top 20-30 holdings with quantities and average buy prices

### Data Preparation
Once data is pulled, compute for every holding:
- Current value = quantity × last_price
- Invested value = quantity × average_price
- P&L = current - invested
- Return % = (last_price - average_price) / average_price × 100

Compute portfolio totals:
- Total invested, total current value, total P&L, overall return %
- Cash balance from margins
- Total wealth = direct equity + MF + cash

---

## Part 1: Deep Portfolio Analysis

> **Do not present this section inline.** Compute all values silently. Everything here feeds directly into the Campfire Dashboard (Part 5), which is the first and only time this data is shown to the user.

### 1.1 Portfolio Snapshot Table

| Metric | Direct Equity | Mutual Funds | Combined |
|--------|--------------|--------------|----------|
| Invested | | | |
| Current Value | | | |
| Unrealised P&L | | | |
| Return % | | | |
| Holdings Count | | | |

### 1.2 Asset Allocation

Classify EVERY holding into asset types:
- Indian Equities (direct + MF equity)
- Gold & Silver (ETFs + SGBs + MF commodity funds)
- International (US/global ETFs + MF international funds)
- Index Trackers (Nifty/Sensex, direct + MF)
- Multi-Asset Funds
- ELSS Tax Saver Funds
- Arbitrage / Liquid
- Debt (if any)
- Cash

### 1.3 Sector Allocation (Direct Equity)

Classify every direct stock into: Banking & Finance, Energy & Utilities, Telecom, IT & Tech, Pharma & Healthcare, Consumer & FMCG, Auto & Industrial, Metals & Mining, Chemicals, Infra & Construction, Travel & Hospitality, Retail & Lifestyle, Conglomerate, Gold/Silver, Index & Liquid ETFs

Show value and % for each sector. Identify top 3 sector bets.

### 1.4 MF Category Allocation

Classify every mutual fund into: Flexi Cap, Nifty 50/Sensex Index, ELSS Tax Saver, Multi-Asset/Dynamic, Silver/Commodity, Arbitrage/Liquid, US/International, Small & Mid Cap, Thematic/Sectoral

Show value and % for each category. Identify AMC concentration (% of MF portfolio per AMC).

### 1.5 Key Tables

- **Top 15 Positions** by current value (stock name, value, return %, P&L)
- **Multibaggers** (100%+ returns) sorted by return %
- **Biggest Losers** sorted by absolute P&L loss

### 1.6 Consolidated Views

Build cross-cutting views that merge direct equity and MF data:
- **Precious Metals (all vehicles)** — every gold/silver instrument across direct + MF
- **International Exposure (all vehicles)** — every US/global instrument
- **Nifty 50 Trackers (all vehicles)** — every Nifty/Sensex instrument

These consolidated views reveal the TRUE allocation that individual portfolio views hide.

### 1.7 Concentration Risk (Direct Equity)

Compute concentration in direct equity holdings:

**Top position concentration:**
```
top1_pct  = largest_holding_value / total_de_value × 100
top3_pct  = sum of top 3 holdings / total_de_value × 100
top5_pct  = sum of top 5 holdings / total_de_value × 100
top10_pct = sum of top 10 holdings / total_de_value × 100
```

**Thresholds:**
- Top 1 position > 15% → flag as HIGH concentration
- Top 3 positions > 35% → flag as HIGH
- Top 5 positions > 50% → flag as MEDIUM
- Top 10 positions > 70% → NORMAL for a focused portfolio

**Stress scenario (compute silently, show as context):**
"If your largest position halved, your direct equity portfolio would drop by X%"
`largest_position_impact = (top1_pct / 2) / 100` → express as % portfolio impact

Always identify: the top 3 single-stock risks by absolute ₹ exposure.

### 1.8 Benchmark Context

Infer approximate portfolio age from multibagger depth — no questions needed:

```
anchor_count_200plus = count of holdings with return > 200%
anchor_count_100plus = count of holdings with return > 100%

if anchor_count_200plus >= 5:
    inferred_age = "5–7 years"
    nifty_cagr_range = "13–15%"
    nifty_total_return_range = "85–150%"
elif anchor_count_100plus >= 5:
    inferred_age = "3–5 years"
    nifty_cagr_range = "13–16%"
    nifty_total_return_range = "44–110%"
elif anchor_count_100plus >= 2:
    inferred_age = "2–3 years"
    nifty_cagr_range = "12–15%"
    nifty_total_return_range = "26–52%"
else:
    inferred_age = "1–2 years"
    nifty_cagr_range = "8–13%"
    nifty_total_return_range = "8–28%"
```

**Nifty 50 approximate CAGR reference (embed these values):**
- 1 year: ~8–12%
- 3 years: ~13–16%
- 5 years: ~14–16%
- 7 years: ~13–15%
- 10 years: ~12–14%
*(These are approximate historical figures — actual returns vary by start/end date)*

**Compute benchmark verdict:**
```
de_return = total direct equity return %

if de_return > top_of_range(nifty_total_return_range):
    verdict = "beating"
    margin = de_return - top_of_range
elif de_return > bottom_of_range(nifty_total_return_range):
    verdict = "roughly matching"
    margin = 0
else:
    verdict = "trailing"
    margin = bottom_of_range - de_return
```

**Always show with caveat:** This is inferred from your multibagger depth, not a true XIRR calculation. True comparison requires transaction dates. Use this as directional context only.

---

### 1.9 Thematic Portfolio Map

Map every holding (direct equity + MF) to one or more themes. This reveals the structural economic bets the investor is making — consciously or not. Themes are cross-sector; a stock can appear in multiple themes.

**India Themes:**
```
India Domestic Consumption    — FMCG, QSR, retail, staples, beverages
India Financial Deepening     — banks, NBFCs, insurance, exchanges, fintech
India Infrastructure Buildout — power, roads, ports, construction, EPC
India Energy Transition       — renewables, green infra, EV
India Travel & Mobility       — aviation, hotels, railways, OTAs
India Digital & Fintech       — internet platforms, SaaS, payments
India Manufacturing / PLI     — auto components, steel, textiles, speciality
India Defence                 — shipbuilding, aerospace, ordnance
India PSU Reform              — state-owned enterprises across all sectors
India Pharma & Chemicals      — formulations, APIs, speciality chemicals, agrochemicals
Precious Metals / Alternatives — gold, silver, SGBs, commodity ETFs
```

**Global Themes (primarily via MFs/ETFs/FOFs):**
```
Global Tech (US)              — Nasdaq 100, S&P 500, MAFANG, US tech stocks
Global Consumption            — international consumer brands (via multi-asset MFs)
Global Emerging Markets       — EM-focused FOFs, China, broad EM
Defensive / Liquid            — arbitrage, liquid, overnight, money market
```

**For each theme, compute:**
- Total ₹ value (direct + MF contributions combined)
- % of total portfolio
- Top 3 holdings within the theme by value
- Whether the theme is reinforced or hedged by MF holdings

**Theme concentration flags:**
- Any single theme > 25% of total portfolio → flag
- Top 2 themes > 45% combined → note the concentration
- Any theme with zero exposure that seems relevant given the portfolio → flag as a gap

**Write a 2-3 sentence thematic portrait** — not a list, a narrative. E.g.: "This portfolio is running three concentrated decade-long theses: India's financial deepening, India's infrastructure buildout, and India's PSU reform story. Global Tech via Nasdaq FOFs adds a US-growth counterweight. India's Defence and Energy Transition themes are conspicuously absent given the strong infrastructure tilt."

**CRITICAL: Derive themes from actual holdings. Do not assign themes generically. Read the specific stocks and funds held.**

---

### 1.10 Expense Ratio Audit

Compute the annual cost of the MF portfolio in rupees.

**For each MF held:**
```
annual_cost_rs = fund_current_value × (expense_ratio / 100)
```

**Expense ratio source (in order of preference):**
1. `actual_expense_ratio` fetched from `mfdata.in/api/v1/schemes/{amfi_code}` — use this when available
2. Approximate ranges for direct plans (fallback when mfdata.in data not available):
   - Active equity funds (flexi cap, large cap, ELSS): 0.5–1.0%
   - Index funds (Nifty 50, LargeMidCap): 0.1–0.2%
   - Multi-asset funds: 0.3–0.5%
   - International FOFs: 0.5–1.0%
   - Silver/Gold ETF FOFs: 0.1–0.3%
   - Arbitrage funds: 0.3–0.5%
   - Liquid funds: 0.1–0.2%

**Compute totals:**
```
total_annual_cost_rs = sum of annual_cost_rs across all MFs
total_annual_cost_pct = total_annual_cost_rs / total_mf_value × 100
```

**Identify savings opportunity:**
For each active equity fund where an equivalent index fund exists:
```
savings_rs = fund_value × (active_er - index_er) / 100
```

Show top 3 funds by annual cost, and total potential savings if switched to cheaper equivalents.

**Example output:** "Your MF portfolio costs approximately ₹X.XL/year in fees (X.X% effective). Your 3 most expensive funds cost ₹X,XXX/year. Switching to index equivalents where available saves ~₹X,XXX/year."

---

### 1.11 Liquidity Profile

Classify every holding by how quickly and cleanly it can be exited.

**Liquidity tiers:**

```
Tier L1 — Highly liquid (T+1, large cap)
  Large cap direct equity (BSE 100 / Nifty 100 names)
  Liquid/arbitrage funds
  Gold ETFs, Silver ETFs
  Index ETFs

Tier L2 — Liquid (T+1 to T+3, may have some slippage)
  Mid cap direct equity
  Active equity MFs (T+3 redemption)
  Multi-asset MFs

Tier L3 — Illiquid (days to weeks, small/micro cap)
  Small cap direct equity
  Less liquid direct equity names

Tier L4 — Locked / restricted
  ELSS funds (3-year lock-in from each SIP date)
  SGBs (8-year maturity, though tradeable)
  Shares under pledge or collateral
```

**Compute:**
```
liquid_value    = L1 + L2 value
illiquid_value  = L3 value
locked_value    = L4 value
liquid_pct      = liquid_value / total_portfolio × 100
```

**Stress scenario:**
"If you needed ₹X.XL urgently (e.g., 25% of portfolio), you could access it from: [list of L1 holdings]. Your ELSS lock-in and SGBs (₹X.XL) cannot be accessed quickly."

Flag if locked_value > 15% of total portfolio — meaningful constraint on liquidity.

---

### 1.12 Dividend & Income Yield

Estimate the annual passive income from the direct equity portfolio.

**Method:** Use approximate trailing dividend yields for dividend-paying stocks. Embed known yields for common dividend payers in Indian markets:

```javascript
const DIVIDEND_YIELDS = {
  // PSU / high-yield names (approx trailing yield)
  "COALINDIA":  6.5, "NTPC": 3.5, "POWERGRID": 4.0,
  "ONGC":       4.5, "IOC":  5.0, "GAIL":      3.5,
  "SBIN":       2.5, "BANKBARODA": 3.0,
  // Consumer / FMCG
  "ITC":        3.5, "HINDUNILVR": 1.5, "BRITANNIA": 1.2,
  "MARICO":     1.5, "DABUR": 1.2,
  // Others
  "HDFCBANK":   1.2, "ICICIBANK": 0.8, "AXISBANK": 0.1,
  "RELIANCE":   0.4, "BHARTIARTL": 0.5,
  // SGBs
  "SGBMAY28-GB": 2.5,  // 2.5% annual interest
};
// Default yield for unlisted stocks: 0%
```

**Compute:**
```
annual_dividend_rs = Σ (holding_value × dividend_yield / 100)
dividend_yield_pct = annual_dividend_rs / total_de_value × 100
```

Show: estimated annual dividend income in ₹, effective yield on direct equity, and top 5 dividend contributors.

Note clearly: "Estimated from trailing yields — actual dividends depend on company decisions and may change."

---

### 1.13 Weighted Portfolio Beta

Estimate directional market sensitivity of the direct equity book.

**Method:** Weight each stock's approximate beta by its portfolio weight.

**Embed approximate betas for common holdings:**
```javascript
const STOCK_BETAS = {
  // High beta (> 1.2) — moves more than market
  "ADANIENT": 1.6, "ADANIGREEN": 1.8, "ADANIPOWER": 1.7,
  "ATHERENERG": 1.9, "TATAPOWER": 1.5, "JSL": 1.4,
  "PAYTM": 1.8, "ETERNAL": 1.6, "SWIGGY": 1.7, "IXIGO": 1.7,
  "DIXON": 1.5, "NEWGEN": 1.4,
  // Near-market beta (0.8–1.2)
  "SBIN": 1.2, "ICICIBANK": 1.1, "HDFCBANK": 0.9,
  "BHARTIARTL": 0.9, "RELIANCE": 1.0, "ITC": 0.7,
  "COALINDIA": 1.0, "NTPC": 0.9, "POWERGRID": 0.8,
  "INDIGO": 1.3, "M&M": 1.1,
  // Low beta (< 0.8) — defensive
  "HINDUNILVR": 0.6, "BRITANNIA": 0.6, "DABUR": 0.6,
  "MARICO": 0.7, "GLAXO": 0.7,
  // Default for unlisted: 1.0
};
```

**Compute:**
```
portfolio_beta = Σ (holding_weight × stock_beta)
  where holding_weight = holding_value / total_de_value
```

**Interpretation:**
- Beta < 0.8 → defensive book, moves less than market
- Beta 0.8–1.2 → market-like
- Beta > 1.2 → aggressive, amplifies market moves in both directions

Show: weighted beta, interpretation, and top 3 highest-beta holdings pulling the number up.

---

## Part 2: Campfire Vitals Report

> **Do not present this section inline.** Compute all 8 scores silently. Output flows exclusively into the Campfire Dashboard (Part 5).

Read the reference file for metric definitions:
→ See `references/vitals-metrics.md`

Score each of the 8 financial health markers on a 0-100 scale. Map them from biological readiness metrics to financial equivalents. Compute and store:
- Each metric name, score, and one-line interpretation
- Campfire Vitals Score (weighted average of all 8 metrics)
- One paragraph summary of financial fitness

**The Campfire Vitals metaphor:** Just as fitness trackers measure biological readiness through HRV, resting heart rate, and sleep quality, Campfire Vitals tracks portfolio readiness through diversification quality, risk calibration, capital efficiency, and more.

---

## Part 3: Investment Style Portrait

> **Do not present this section inline.** Write the paragraph internally. It is presented once, inside the Campfire Dashboard (Part 5).

Write a single rich paragraph (150-250 words) that captures:

1. **Primary investing style** — what kind of investor is this person? (Not a label — a description of HOW they invest based on observable behavior)
2. **Strengths** — what the portfolio reveals they're good at
3. **Blind spots** — what structural gaps or biases the data reveals
4. **The contradiction** — almost every portfolio has one (e.g., "researches stocks deeply but fragments gold across 7 vehicles without consolidating")
5. **The signature move** — their most distinctive pattern (e.g., "accumulates winners quietly" or "IPO hunter who holds through drawdowns")

**CRITICAL: Write this from the data, not from assumptions. Read the actual holdings, the actual returns, the actual allocation. Two portfolios with the same total value can reveal completely different personalities.**

---

## Part 4: Portfolio X-Ray

> **Do not present this section inline.** Compute the fund classification silently. The actual API calls happen in the user's browser via JavaScript — NOT from Claude's container. Claude's job is to generate correct JavaScript that fetches on page load in the background, so combined concentration data is ready by the time the user reads the Analysis tab.

This is Campfire Analyst's deepest analysis. It classifies every MF held, then generates a JavaScript engine inside the dashboard HTML that fetches mfdata.in **immediately on page load** and feeds the results into two places:
1. **The Concentration Risk section of the Analysis tab** — updated with combined direct + MF exposure
2. **The X-Ray tab** — the detailed breakdown (fund holdings, overlap map, asset class summary)

Read the reference file for full spec:
→ See `references/xray-template.md`

### Architecture: Background Fetch on Page Load

**CRITICAL: mfdata.in is fetched via JavaScript in the user's browser — not by Claude.**
Claude's container cannot reach mfdata.in. The user's browser can.

**CRITICAL: The fetch starts immediately when the HTML file opens — not when the user clicks the X-Ray tab.** This ensures concentration data is combined and ready when the user views the Analysis tab.

Claude's job in Part 4:
1. Classify each MF fund into the four-tier coverage model
2. Embed the classification + fund data as a JavaScript array in the HTML
3. The JavaScript engine fetches on page load (background, non-blocking), then:
   - Updates the Concentration Risk section in the Analysis tab with combined numbers
   - Populates the X-Ray tab with full detail

### Four-Tier Coverage Model

Classify every MF fund into one of four tiers. Each tier gets different treatment:

**Tier 1 — Live stock look-through**
Funds: `active_equity` (flexi cap, large cap, ELSS, multi-asset, thematic)
Also try: `index` funds — attempt mfdata.in first, use if data returns
Action: JavaScript fetches `mfdata.in/api/v1/families/{id}/holdings` at runtime
Output: Full stock-by-stock look-through with ₹ values

**Tier 2 — Approximate index look-through**
Funds: `index` funds where Tier 1 fetch fails (NOT_FOUND from mfdata.in)
Action: Use embedded approximate top-weight data for major indices:
- Nifty 50 top holdings: HDFCBANK ~13%, ICICIBANK ~8%, RELIANCE ~8%, INFY ~6%, TCS ~5%, BHARTIARTL ~4%, HINDUNILVR ~3%, BAJFINANCE ~3%, KOTAKBANK ~3%, LT ~3%
- Nifty LargeMidCap 250 / Nifty Smallcap 250: Attribute to "Index — constituents not individually mapped"
Output: Approximate top-10 stock look-through, labelled "Approximate — based on index composition"

**Tier 3 — Asset class attribution**
Funds: Commodity FOFs (gold/silver), international FOFs (US equity)
Action: No stock-level data. Count fund value toward the correct asset class:
- Gold ETF FOF, Gold FOF → gold exposure
- Silver ETF FOF → silver exposure
- Multi-asset FOF → attempt mfdata.in; if fails, use known asset split from fund factsheet (e.g. Zerodha Multi Asset ≈ 65% equity + 17.5% gold + 17.5% silver)
- Nasdaq 100 FOF, S&P 500 FOF → international/US equity exposure
Output: Shows correctly in Asset Class Look-Through Summary, labelled "Via FOF"

**Tier 4 — Cash equivalent**
Funds: Arbitrage, liquid, overnight
Action: No look-through. Show as cash buffer in asset class summary.
Output: Correctly attributed in allocation, not in stock table

### Classification Keywords

```
Tier 1 active_equity:  Flexi Cap, Large Cap, ELSS, Multi Asset Allocation,
                        Thematic, Sectoral, Small Cap, Mid Cap, Balanced

Tier 1/2 index:        "Nifty", "Sensex", "Index Fund", "Index ETF",
                        "Momentum", "Smallcap 250", "LargeMidcap", "Equal Weight"

Tier 3 commodity_fof:  "Gold ETF", "Silver ETF", "Gold BeES", "Silver BeES",
                        "Gold FOF", "Silver FOF", "Commodity"

Tier 3 international:  "Nasdaq", "S&P 500", "SP 500", "US Equity",
                        "Global", "International", "Overseas"

Tier 3 multi_asset_fof: "Multi Asset" + "FOF" or "Passive FOF" or "FoF"

Tier 4 arbitrage:      "Arbitrage", "Liquid", "Overnight", "Money Market",
                        "Ultra Short"
```

### What Claude Generates

Claude outputs in the X-Ray tab JavaScript:
```javascript
const MF_FUNDS = [
  // Tier 1 — active equity / multi-asset (live fetch from mfdata.in)
  // sharpe_3y, sortino_3y, alpha_3y, beta_3y, actual_expense_ratio populated at runtime by Step 2 fetch
  {name:"FUND NAME", search:"search query", value:NNNN, tier:1, type:"active_equity"},

  // Tier 1/2 — index fund (try mfdata.in, fall back to embedded weights)
  {name:"FUND NAME", search:"", value:NNNN, tier:2, type:"index", indexName:"Nifty 50"},

  // Tier 3 — commodity / international FOF (asset class only)
  {name:"FUND NAME", search:"", value:NNNN, tier:3, type:"commodity_fof", assetClass:"silver"},

  // Tier 4 — arbitrage / liquid (cash equivalent)
  {name:"FUND NAME", search:"", value:NNNN, tier:4, type:"arbitrage"},
];
```

The JavaScript engine:
1. Fetches holdings from `mfdata.in/api/v1/families/{family_id}/holdings`
2. Fetches ratios from `mfdata.in/api/v1/families/{family_id}/ratios` → populates `sharpe_3y`, `sortino_3y`, `alpha_3y`, `beta_3y`
3. Fetches scheme details from `mfdata.in/api/v1/schemes/{amfi_code}` → populates `actual_expense_ratio`
4. Updates Concentration Risk section in Analysis tab with combined numbers
5. Populates the X-Ray tab: look-through table + Fund Quality Summary (Sharpe, Sortino, Alpha, expense, verdict)

See `references/xray-template.md` for the full JavaScript spec and fallback handling.

---

## Part 5: Campfire Dashboard

**This is the first thing the user sees — and the only place Parts 1-4 are presented.** Do not summarise or preview any analysis before generating this. Go straight from data-pulling to this file.

Generate a **single self-contained HTML file** with four tabs, organised by decision urgency — not output type:

```html
<nav>
  <button class="tab active">Summary</button>
  <button class="tab">Fix List</button>
  <button class="tab">Analysis</button>
  <button class="tab">X-Ray</button>
</nav>
```

Read the reference files for full design specs:
→ `references/dashboard-template.md` — design tokens, layout, shared CSS
→ `references/task-center-template.md` — Fix List tab
→ `references/xray-template.md` — X-Ray tab

---

### Tab 1: Summary

**Answers: "Am I OK?" — fast read, no scrolling required.**

**1. Headline Finding**
One bold sentence. The single most important insight from the entire analysis.
Use this priority order to pick it:

```
1. Any single direct equity position > 15% of direct equity book
   → "[Stock] is X% of your direct equity — if it halved, your portfolio drops ₹Y.YL"
   (Note: combined exposure including MFs will be shown in Analysis tab once loaded)

2. Any security with effective look-through exposure > 10% of total portfolio (combined)
   → "[Stock] is your largest real position at X% combined — direct + N funds"

3. Any High Impact + Quick Win task
   → "You have N Nifty 50 funds doing the same job — consolidating saves ₹X.XL in complexity"

4. More than 5 multibaggers
   → "Your stock picks are working — N positions have doubled or more"

5. Largest fragmentation issue
   → "X gold instruments, Y ELSS funds — one each would do the same job"

6. Return vs estimated Nifty 50 benchmark
   → "Your portfolio is [beating / trailing] the Nifty 50 benchmark"
```

Style: 18-20px, font-weight 600, `#37352f`. No icon, no colour. Just the sentence.

**2. Hero Numbers**
Three cards: Total Wealth · Overall Return % · Holdings Count

**3. Wealth Composition Bar**
Horizontal stacked bar: Direct Equity vs. MF vs. Cash.
Show ₹ value and return % for each segment below the bar.

**4. Top 3 Quick Wins**
From the task list, pick the 3 tasks where `impact = "high"` and `effort = "low"`.
Show as simple rows — task name, one-line description, ₹ value addressable.
Each row is clickable — clicking opens the Fix List tab with that task expanded.

**5. Multibaggers & Losers**
Top 5 multibaggers and top 5 losers, side by side.

---

### Tab 2: Fix List

**Answers: "What should I do?" — the most actionable tab.**

Full task center as per `references/task-center-template.md`. No changes to task logic or design.
Implement in vanilla JS + HTML. All state in JS variables.

---

### Tab 3: Analysis

**Answers: "Why does my portfolio look this way?" — for the curious.**

**Self-assessment**
1. **Campfire Vitals** — all 8 scores with progress bars and one-line interpretations. Overall score.
2. **Investment Style Portrait** — the paragraph from Part 3.

**What am I actually betting on?**
3. **Thematic Map** — from Part 1 section 1.9.
   - Horizontal bars showing ₹ value and % per theme
   - 2-3 sentence thematic portrait (narrative, not list)
   - Theme gaps flagged: relevant themes with zero exposure

**How am I performing?**
4. **Benchmark Context** — from Part 1 section 1.8. Inferred portfolio age, verdict vs Nifty 50.

**What are my risks?**
5. **Concentration Risk** — single unified view combining direct equity + MF look-through.
   - Shows combined effective exposure per security (direct + via all MFs)
   - Top 1 / Top 3 / Top 5 as % of total portfolio — not just direct equity
   - Flags if any security > 10% of total portfolio (combined)
   - Halving scenario: "If [largest combined position] halved, your total portfolio drops ₹Y.YL"
   - Starts with direct-only numbers on page load. Updates automatically once X-Ray fetch completes.
   - Label: "Direct equity only" until updated, then "Direct + MF look-through (as of [month])"

**Portfolio health at a glance**
6. **Portfolio Metrics** — four compact metric cards in a 2×2 grid:
   - **Expense Cost** — total annual MF fees in ₹/year (from 1.10)
   - **Liquidity** — % of portfolio accessible quickly (from 1.11)
   - **Dividend Yield** — estimated annual income from direct equity (from 1.12)
   - **Portfolio Beta** — weighted beta of direct equity book (from 1.13)

**The raw data**
7. **Top 10 Holdings** — merged list across direct equity and MF, sorted by current value.
8. **Sector Allocation** — horizontal bars for direct equity sectors.

---

### Tab 4: X-Ray

**Answers: "What's inside each of my funds?" — the supporting detail.**

The X-Ray tab shows the detail behind the Concentration Risk numbers in the Analysis tab:
- Fund-by-fund breakdown — what each fund holds and at what weight
- Overlap map — which fund pairs hold the same stocks
- Asset class look-through — true allocation across equity/debt/gold/international/cash
- Coverage summary — which funds were fetched live, which used approximate data, which are cash equivalents

Data loads in the background on page open — by the time the user clicks this tab, it should already be populated. If still loading, show a progress indicator.

See `references/xray-template.md` for full spec.

---

### Shared Design

All four tabs share the same design tokens:

```css
--bg: #ffffff;
--surface: #f7f7f5;
--border: #e8e8e4;
--text: #37352f;
--text-secondary: #787774;
```

Notion-like minimalist. White background, Inter font, thin dividers. No dark theme, no shadows, no gradients. Max-width 720px, centered. Self-contained HTML with inline CSS and inline JS.

Save to `/mnt/user-data/outputs/campfire-dashboard.html` and present to user.

---

## Part 6: investment.md Generation

Compile Parts 1-3 into a structured markdown document with these sections:

1. Combined Wealth Overview (the master table)
2. Asset Allocation (full portfolio, merged direct + MF)
3. Direct Equity Portfolio (snapshot, sectors, top positions, baggers, losers)
4. Mutual Fund Portfolio (overview, categories, top holdings, AMC concentration)
5. Consolidated Views (precious metals, international, Nifty trackers — with fragmentation notes)
6. Thematic Portfolio Map (₹ and % per theme, thematic portrait, theme gaps)
7. Portfolio Metrics (expense cost, liquidity profile, dividend yield, weighted beta)
8. Investor Identity Profile (style tags, behavioral signals, consumer personality type)
9. Portfolio Gaps & Opportunities (table of gaps with current state + suggestion)

Save to `/mnt/user-data/outputs/investment.md` and present to user.

**DO NOT include any personal details (name, city, property, phone model, car, etc.) in investment.md. It should contain ONLY what the portfolio data itself reveals. Personal details belong in taste.md after the prediction game validates them.**

---

## Part 7: taste.md Consumer Identity

**This is the interactive, conversational part. It should feel like a game, not a report.**

### 7.1 Extract the Personality Archetype

Read the reference file for archetype definitions:
→ See `references/archetypes.md`

Analyze the portfolio for 6 behavioral signals:

1. **Holding Period** — average prices vs. current prices across portfolio
2. **Concentration vs. Diversification** — number of holdings, top 5 concentration
3. **Risk Appetite** — F&O usage, small cap %, gold/defensive allocation
4. **Value System** — PSU vs. new-age vs. MNC mix
5. **Decision Style** — ETF usage, consolidation patterns, portfolio cleanliness
6. **Income and Life Stage** — portfolio size, dividend vs. growth tilt

Map to archetype(s). Most people are 2-3 archetypes blended. Identify primary + secondary.

### 7.2 The Prediction Game

Read the reference file for prediction methodology:
→ See `references/prediction-chains.md`

Present predictions ONE AT A TIME, starting with highest confidence domains:

**Sequence:**
1. Phone (brand, model tier, age)
2. Car (brand, category, age)
3. Watch (brand, price tier — or no watch)
4. TV (brand, size)
5. → After 4-5 predictions, ASK: "Where are you originally from? Where do you live now?"
6. Clothing style
7. Hotel preference
8. Music app
9. Banking
10. Travel destinations
11. Cultural preferences (with origin/city data now available)

**For each prediction:**
- State the prediction
- Show the reasoning chain (which portfolio signal → which behavioral inference → which prediction)
- Wait for user to confirm or correct
- If wrong: ask "What's the actual answer? What made you choose that?" then classify the miss

**NEVER:**
- Default to any archetype's "typical" predictions
- Assume prior users' answers apply
- Skip showing reasoning
- Batch predictions together (always one at a time)

**ALWAYS:**
- Let the data lead — extract signals first, form archetype second, predict third
- Predict the TOP VARIANT within their tier
- Celebrate misses as discoveries about what data alone can't capture
- Check consistency across predictions (price tier, brand philosophy, hold period should align)

### 7.3 Generate taste.md

After the prediction game, compile:

1. Portfolio Summary — 2-3 lines only: total wealth, overall return %, asset split. **Do not copy investment.md sections here** — link readers to that file for the full data picture.
2. Consumer Personality Type (archetype name + description)
3. The Decision Matrix (all predictions, hits and misses)
4. Operating Modes (how this specific person makes decisions differently across domains)
5. Predictive Signals for Agents (what converts them, what doesn't)
6. Prediction Scorecard (accuracy %, miss classification)
7. What Different Agents Would Learn (wealth advisor, lender, brand, real estate agent)
8. Cohort Estimate

Save to `/mnt/user-data/outputs/taste.md` and present to user.

---

## Execution Sequence

When triggered, run in this order:

1. **Pull data** (Step 0) — get holdings, MF, margins
2. **Compute silently** (Parts 1-3) — run all analysis, score Campfire Vitals, write style portrait, generate task list. **Do not output anything to the user yet.**
3. **Portfolio X-Ray classification** (Part 4) — classify every MF fund into the four-tier coverage model. Prepare the JavaScript fund data array for the dashboard. **The JavaScript fetches mfdata.in on page load in the background — feeding both the Concentration Risk section of the Analysis tab and the X-Ray tab.**
4. **Campfire Dashboard** (Part 5) — generate single four-tab HTML file (Summary + Fix List + Analysis + X-Ray), save, present. **This is the first thing the user sees.**
5. **Generate investment.md** (Part 6) — save the full wealth document, present to user
6. **taste.md Prediction Game** (Part 7) — interactive, takes multiple turns

Steps 1-4 run in a single response. Part 6 follows immediately after. Part 7 requires back-and-forth conversation.

If the user asks for a specific part only (e.g., "just give me my Campfire Vitals report", "run Portfolio X-Ray", or "generate my taste.md"), jump to that section directly — but always pull data first.

---

## Key Principles

### Zero Prior Bias
Every run starts fresh. No default archetype. No assumed preferences. No carried-over predictions. The data speaks.

### Dual Strategy Detection
Always check for the dual strategy pattern: active direct equity + passive MF. Many Indian investors run both. Neither portfolio alone tells the full story. Always merge for the true picture.

### Fragmentation Awareness
Look for the accumulation-without-consolidation pattern: same asset class across many vehicles (N Nifty funds, N gold ETFs). This is extremely common and is a key cleanup opportunity.

### Honest Confidence Levels
- Purchase decisions (phone, car, watch, TV): 70-90% accuracy from portfolio alone
- Lifestyle systems (hotel, airline, bank): 50-70% accuracy
- Cultural preferences (food, entertainment, travel): 30-50% accuracy — always caveat these and ask for origin/city data

### The File Hierarchy
- **campfire-dashboard.html** = the consolidated app (Summary → Fix List → Analysis → X-Ray — first thing the user sees)
- **investment.md** = the wealth document (pure data, no personal predictions)
- **taste.md** = the identity document (predictions, validated through conversation)

The dashboard gives the full picture in one place. investment.md goes deep. taste.md gets personal.
