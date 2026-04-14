# Campfire Portfolio X-Ray — Template & Design Spec

## Overview

The Portfolio X-Ray is the **fourth tab of `campfire-dashboard.html`**. It is the **detail view** behind the Concentration Risk numbers shown in the Analysis tab.

**Separation of concerns:**
- **Analysis tab → Concentration Risk section** = the conclusion ("You are 8.4% in HDFC Bank combined")
- **X-Ray tab** = the evidence ("Here's how that breaks down across PPFAS, DSP, and direct equity")

The user gets one answer to "am I overweight?" in one place. The X-Ray tab is for those who want to understand why.

**Architecture: Background fetch on page load.** The JavaScript engine starts fetching mfdata.in immediately when the HTML file opens — not when the user clicks the X-Ray tab. This ensures:
1. Concentration Risk in the Analysis tab has combined numbers ready by the time the user reads it
2. The X-Ray tab is already populated when clicked — no waiting

**Coverage model: Four tiers.** Target: ~85% stock-level, ~100% asset-class-level.

---

## Fetch Trigger

```javascript
// Runs immediately on page load — not on tab click
document.addEventListener('DOMContentLoaded', () => {
  loadXray(); // fetch all Tier 1 funds in background
});

async function loadXray() {
  // ... fetch mfdata.in for each Tier 1/2 fund
  // When complete: updateConcentrationRisk(allExposure);  ← updates Analysis tab
  //                renderXrayTab(allExposure);             ← populates X-Ray tab
}
```

The Analysis tab's Concentration Risk section starts with direct-equity-only numbers (computed by Claude at generation time). When `loadXray()` completes, it calls `updateConcentrationRisk()` to replace those numbers with the combined direct + MF view.

The X-Ray tab shows a loading indicator ("Fetching fund data...") until `renderXrayTab()` is called.

---

## Four-Tier Coverage Model

### Tier 1 — Live stock look-through
**Funds:** `active_equity` (flexi cap, large cap, ELSS, multi-asset, thematic) + `index` funds (attempt first)
**JavaScript action:** Search mfdata.in for `family_id`, then fetch `/families/{id}/holdings`
**Output:** Full stock-by-stock look-through with ₹ values and sectors
**Label in UI:** "✓ Live data from mfdata.in — [month] disclosure"

### Tier 2 — Approximate index look-through
**Funds:** `index` funds where Tier 1 returns `NOT_FOUND`
**JavaScript action:** Use embedded approximate top-weight data for the specific index
**Nifty 50 approximate weights (top 10):**
```javascript
const NIFTY50_APPROX = [
  {name:"HDFCBANK", sector:"Financial Services", weight_pct:13.0},
  {name:"ICICIBANK", sector:"Financial Services", weight_pct:8.2},
  {name:"RELIANCE",  sector:"Energy",             weight_pct:8.0},
  {name:"INFY",      sector:"Technology",          weight_pct:6.1},
  {name:"TCS",       sector:"Technology",          weight_pct:5.2},
  {name:"BHARTIARTL",sector:"Communication",       weight_pct:4.1},
  {name:"HINDUNILVR",sector:"Consumer Defensive",  weight_pct:3.2},
  {name:"BAJFINANCE",sector:"Financial Services",  weight_pct:3.1},
  {name:"KOTAKBANK", sector:"Financial Services",  weight_pct:3.0},
  {name:"LT",        sector:"Industrials",         weight_pct:2.9},
  // remaining ~43% spread across 40 stocks — attribute to "Other Nifty 50 constituents"
];
```
**Output:** Approximate top-10 look-through, clearly labelled "Approximate — top 10 of Nifty 50 index"
**Label in UI:** "◎ Approximate — Nifty 50 top holdings (index fund)"

For other indices (LargeMidCap 250, Smallcap 250, Momentum 30): attribute total value to "Index fund — constituents not individually mapped" in the asset class summary.

### Tier 3 — Asset class attribution
**Funds:** Commodity FOFs (gold/silver), international FOFs (US equity), multi-asset passive FOFs
**JavaScript action:** No stock-level data. Count toward correct asset class in the summary.

Asset class mapping:
```javascript
const TIER3_MAPPING = {
  gold_fof:        {assetClass:"Gold",          category:"Commodity"},
  silver_fof:      {assetClass:"Silver",         category:"Commodity"},
  nasdaq_fof:      {assetClass:"US Equity",      category:"International"},
  sp500_fof:       {assetClass:"US Equity",      category:"International"},
  us_equity_fof:   {assetClass:"US Equity",      category:"International"},
  multi_asset_fof: {assetClass:"Multi-Asset",    category:"Mixed"},
};
```

For multi-asset FOFs: attempt Tier 1 fetch first. If holdings return, use them. If not, use the fund's disclosed approximate split (typically shown in fund factsheet) to attribute to equity/gold/silver.

**Output:** Shows in Asset Class Look-Through Summary table only, not in the stock table
**Label in UI:** "○ Via FOF — asset class attribution only"

### Tier 4 — Cash equivalent
**Funds:** Arbitrage, liquid, overnight, money market
**JavaScript action:** None. Attribute to cash/liquid in asset class summary.
**Output:** Shows in Asset Class Look-Through Summary as "Arbitrage/Liquid"
**Label in UI:** "○ Cash equivalent — no equity look-through"

---

## Data Pipeline

**Primary API: `mfdata.in`** — free, no auth. Fetched by browser JavaScript at runtime.
**Fallback:** Embedded approximate index weights for Tier 2.
**Data freshness:** AMFI monthly disclosures — up to 30 days old. Always shown in tab header.

### Step 1 — Fund classification (Claude does this, embeds in JS)

Claude classifies every MF held into a tier and type:

```javascript
// Claude generates this array based on the user's actual MF holdings
const MF_FUNDS = [
  {name:"FUND NAME", search:"search query for mfdata.in", value:NNNN,
   tier:1, type:"active_equity"},
  {name:"FUND NAME", search:"Nifty 50 Index Direct",      value:NNNN,
   tier:1, type:"index", indexName:"Nifty 50", indexApprox: NIFTY50_APPROX},
  {name:"FUND NAME", search:"",                           value:NNNN,
   tier:3, type:"gold_fof",    assetClass:"Gold"},
  {name:"FUND NAME", search:"",                           value:NNNN,
   tier:3, type:"nasdaq_fof",  assetClass:"US Equity"},
  {name:"FUND NAME", search:"",                           value:NNNN,
   tier:4, type:"arbitrage"},
];
```

Classification keyword rules (check fund name):
- `"FOF"` or `"Fund of Fund"` or `"Passive FoF"` or `"ETF FoF"` → FOF subtype (then check what it holds)
- `"Nasdaq"` or `"S&P 500"` or `"SP 500"` or `"US Equity"` or `"Overseas"` → tier 3, `nasdaq_fof` or `us_equity_fof`
- `"Gold ETF"` or `"Gold FoF"` or `"Gold BeES"` → tier 3, `gold_fof`
- `"Silver ETF"` or `"Silver FoF"` or `"Silver BeES"` → tier 3, `silver_fof`
- `"Multi Asset"` + (`"FOF"` or `"Passive"`) → tier 3, `multi_asset_fof` (attempt Tier 1 first)
- `"Nifty"` or `"Sensex"` or `"Index Fund"` or `"Momentum"` or `"Equal Weight"` → tier 1/2, `index`
- `"Arbitrage"` or `"Liquid"` or `"Overnight"` or `"Money Market"` → tier 4, `arbitrage`
- Everything else → tier 1, `active_equity`

### Step 2 — JavaScript fetches (browser, at runtime)

For each Tier 1 fund, three fetches in sequence:

```javascript
// 1. Search for family_id and amfi_code
const sr = await fetch(`https://mfdata.in/api/v1/search?q=${encodeURIComponent(fund.search)}`);
const sd = await sr.json();
const schemes = sd.data || [];
const direct = schemes.find(s => s.plan_type === 'direct') || schemes[0];
const family_id = direct.family_id;
const amfi_code = direct.amfi_code;

// 2. Fetch holdings (confirmed endpoint)
const hr = await fetch(`https://mfdata.in/api/v1/families/${family_id}/holdings`);
// Returns equity_holdings[], debt_holdings[], other_holdings[]
// Each item has: stock_name / name, weight_pct, sector, market_value

// 3. Fetch ratios + actual expense ratio (parallel)
// Ratios endpoint — assumed pattern, confirmed Sharpe/Beta/PE data exists on mfdata.in
// Falls back gracefully if endpoint returns 404 or empty
let sharpe_3y = null, sortino_3y = null, alpha_3y = null, beta_3y = null, actual_expense_ratio = null;
try {
  const [ratiosRes, schemeRes] = await Promise.all([
    fetch(`https://mfdata.in/api/v1/families/${family_id}/ratios`),
    fetch(`https://mfdata.in/api/v1/schemes/${amfi_code}`)
  ]);

  if (ratiosRes.ok) {
    const rd = await ratiosRes.json();
    const r = rd.data;
    if (r) {
      sharpe_3y  = r.sharpe_3y  ?? r.sharpe  ?? null;
      sortino_3y = r.sortino_3y ?? r.sortino ?? null;
      alpha_3y   = r.alpha_3y   ?? r.alpha   ?? null;
      beta_3y    = r.beta_3y    ?? r.beta    ?? null;
    }
  }

  if (schemeRes.ok) {
    const sc = await schemeRes.json();
    actual_expense_ratio = sc.data?.expense_ratio ?? null;
  }
} catch(e) {
  // ratios or scheme fetch failed — continue without them
}

// Store per fund
fund.sharpe_3y            = sharpe_3y;
fund.sortino_3y           = sortino_3y;
fund.alpha_3y             = alpha_3y;
fund.beta_3y              = beta_3y;
fund.actual_expense_ratio = actual_expense_ratio;
```

**Note on ratios endpoint:** The URL `mfdata.in/api/v1/families/{id}/ratios` follows the same pattern as holdings and is consistent with mfdata.in's documented Sharpe/Beta/PE feature. If the endpoint structure differs when tested live, update the field names accordingly. The `try/catch` ensures the rest of the X-Ray continues working even if ratios are unavailable.

**Scheme endpoint is confirmed:** `mfdata.in/api/v1/schemes/{amfi_code}` returns `expense_ratio` directly — use this instead of the approximate ranges in section 1.10 whenever available.

If NOT_FOUND on holdings and fund type is `index` → fall back to Tier 2 (use embedded index weights).
If NOT_FOUND on holdings and fund type is `active_equity` → mark `data_unavailable`.

### Step 3 — Look-through calculation (in browser JavaScript)

```javascript
// For each security across all Tier 1 + Tier 2 funds:
look_through_value = fund.value × (weight_pct / 100);

// Aggregate:
total_look_through[security] += look_through_value;

// Add direct equity:
effective_exposure[security] = total_look_through[security] + direct_holding_value;
effective_pct = effective_exposure[security] / total_portfolio_value × 100;
```

### Step 4 — Update Analysis tab (combined concentration)

After aggregating all look-through values, call `updateConcentrationRisk(allExposure)`:

```javascript
function updateConcentrationRisk(allExposure) {
  // Sort by effective exposure descending
  const sorted = Object.entries(allExposure)
    .map(([name, d]) => ({name, ...d, pct: d.total / TOTAL_PORTFOLIO * 100}))
    .sort((a, b) => b.total - a.total);

  // Update top-N concentration bars in Analysis tab
  const top1 = sorted[0]?.pct || 0;
  const top3 = sorted.slice(0,3).reduce((s,h) => s + h.pct, 0);
  const top5 = sorted.slice(0,5).reduce((s,h) => s + h.pct, 0);

  // Replace "direct only" placeholders with combined values
  document.getElementById('conc-top1').textContent = top1.toFixed(1) + '%';
  document.getElementById('conc-top3').textContent = top3.toFixed(1) + '%';
  document.getElementById('conc-top5').textContent = top5.toFixed(1) + '%';
  document.getElementById('conc-source-label').textContent =
    'Direct + MF look-through (as of [month])';

  // Update halving scenario
  const largest = sorted[0];
  if (largest) {
    const halvingImpact = (largest.total / 2 / TOTAL_PORTFOLIO * 100).toFixed(1);
    const halvingRs = (largest.total / 2 / 100000).toFixed(2);
    document.getElementById('conc-halving').textContent =
      `If ${largest.name} halved, your portfolio drops ${halvingImpact}% (₹${halvingRs}L)`;
  }

  // Flag hidden concentrations (> 10% of total portfolio)
  const flags = sorted.filter(h => h.pct > 10 && h.direct === 0);
  // Render flags in the concentration section
  renderHiddenConcentrationFlags(flags);
}
```

Key principle: **the Analysis tab shows the conclusion, the X-Ray tab shows the breakdown.** `updateConcentrationRisk` only updates summary numbers. The full security-by-security table stays in the X-Ray tab.

### Output 1: Consolidated Look-Through Table

Every unique security across all funds + direct equity, sorted by effective exposure descending.

| Security | Type | Direct | Fund A | Fund B | Fund C | Look-Through Total | Effective % |
|---|---|---|---|---|---|---|---|
| HDFC Bank | Equity | ₹2.1L | 8.2% → ₹1.4L | 9.4% → ₹0.8L | — | ₹4.3L | 4.3% |
| Infosys | Equity | — | 4.1% → ₹0.7L | 6.2% → ₹0.5L | 3.8% → ₹0.4L | ₹1.6L | 1.6% |
| Govt Bond 2034 | Debt | — | — | 4.1% → ₹0.3L | 6.8% → ₹0.6L | ₹0.9L | 0.9% |

Columns:
- **Security** — name of stock, bond, or instrument
- **Type** — Equity / Debt / Gold / Other
- **Direct** — direct holding value if any (else —)
- **Fund columns** — one column per MF held, showing weight% → ₹ value (show — if fund doesn't hold it)
- **Look-Through Total** — sum of all fund exposure + direct
- **Effective %** — as % of total portfolio value

### Output 1b: Fund Quality Summary

For each Tier 1 fund where ratios were successfully fetched, show a compact quality row:

| Fund | Return | Sharpe (3Y) | Sortino (3Y) | Alpha (3Y) | Expense | Verdict |
|---|---|---|---|---|---|---|
| PPFAS Flexi Cap | +11.0% | 1.2 | 1.5 | +2.1% | 0.58% | ✓ High quality |
| Capitalmind Flexi Cap | +8.2% | 0.5 | 0.6 | -1.4% | 0.94% | ⚠ Underperforming |
| DSP Multi Asset | -1.4% | 0.3 | 0.2 | -3.8% | 0.35% | ✗ Exit candidate |

**Ratio interpretation:**

Sharpe (3Y) — return per unit of total risk:
- ≥ 1.0 → green — good risk-adjusted return
- 0.5–1.0 → amber — moderate
- < 0.5 → red — poor

Sortino (3Y) — return per unit of downside risk only:
- ≥ 1.5 → green — strong downside protection
- 0.8–1.5 → amber — acceptable
- < 0.8 → red — poor downside management
- Note: Sortino is always ≥ Sharpe for a given fund. A large gap (e.g. Sharpe 0.8, Sortino 1.4) means the fund's volatility is mostly upside — good sign. A small gap (Sharpe 0.8, Sortino 0.9) means most volatility is downside — bad sign.

Alpha (3Y) — excess return over benchmark after adjusting for beta:
- > 0 → green — manager adding value above market exposure
- -1% to 0 → amber — roughly market-like after risk adjustment
- < -1% → red — manager destroying value; strong case for switching to index

**Verdict logic:**
- ✓ High quality: Sharpe ≥ 1.0 AND Alpha > 0 (fee justified by skill + efficiency)
- ⚠ Review: any ratio in amber range, or Sharpe/Sortino conflict
- ✗ Exit candidate: Alpha < -1% AND Sharpe < 0.5 (paying for underperformance)

This table feeds directly into the task generator for consolidation and switch recommendations.

### Output 2: Hidden Concentration Flags

Any security where effective exposure exceeds **5% of total portfolio** — especially if the user didn't consciously choose it.

```
⚠️ HDFC Bank — 6.8% effective exposure
   Direct: ₹2.1L | Via 3 funds: ₹4.7L | Total: ₹6.8L
   You own this across: Direct Equity, PPFAS Flexi Cap, Mirae Large Cap

⚠️ Reliance Industries — 5.4% effective exposure
   Direct: ₹1.4L | Via 4 funds: ₹4.0L | Total: ₹5.4L
   You own this across: Direct Equity, Axis Bluechip, Nifty 50 ETF x2
```

Threshold: Flag if effective % > 5%. Warn if 3-5%. Green if < 3%.

### Output 3: Fund Overlap Map

For each pair of funds held, calculate overlap score:
```
overlap = sum of min(weight_in_fund_A, weight_in_fund_B) for all shared securities
```

Present as a simple matrix or list:

```
PPFAS Flexi Cap ↔ Mirae Large Cap     62% overlap — high redundancy
PPFAS Flexi Cap ↔ Axis Bluechip       48% overlap — moderate redundancy
Mirae Large Cap ↔ Axis Bluechip       71% overlap — very high redundancy
```

Flag pairs with > 50% overlap as candidates for consolidation.

### Output 4: Asset Class Look-Through Summary

Break down effective portfolio by true asset class after look-through:

| Asset Class | Direct | Via MFs | Total | % of Portfolio |
|---|---|---|---|---|
| Indian Equity | ₹X | ₹X | ₹X | X% |
| US/International Equity | — | ₹X | ₹X | X% |
| Debt | — | ₹X | ₹X | X% |
| Gold & Silver | ₹X | ₹X | ₹X | X% |
| Cash & Liquid | ₹X | ₹X | ₹X | X% |

This is the TRUE allocation — not what the top-level MF labels suggest.

---

## Tab Design

### Visual Design

Same Notion-like design tokens as Summary, Fix List, and Analysis tabs:
- Background: `#ffffff`
- Surface: `#f7f7f5`
- Border: `#e8e8e4`
- Text: `#37352f`
- Secondary: `#787774`
- Font: Inter

### Tab Header

```
Fund Detail View                   Data as of: [latest AMFI disclosure date]
⚠️ AMFI monthly disclosures — data may be up to 30 days old
```

A brief orientation note at the top:
```
These are the holdings behind your Concentration Risk numbers in the Analysis tab.
[N] funds loaded · [N] approximate · [N] asset class only · [N] cash equivalent
```

### Section Order

1. **Asset Class Look-Through Summary** — the corrected true allocation (most impactful, show first)
2. **Hidden Concentration Flags** — ⚠️ warnings for positions > 5%
3. **Consolidated Look-Through Table** — full sortable table (default sort: effective % desc)
4. **Fund Overlap Map** — which funds are redundant

### Table Interaction (vanilla JS)

- Click column header to sort (effective %, security name, type)
- Filter buttons: All / Equity / Debt / Gold
- Rows with direct + look-through exposure highlighted with subtle left border in `#6940a5`
- Concentration flags inline in the table (red dot for > 5%, amber for 3-5%)

### Flags Design

Use the same pill/badge system as Task Center:
```css
.flag-high   { color: #e03e3e; background: #fbe4e4; } /* > 5% */
.flag-medium { color: #d9730d; background: #fdecc8; } /* 3-5% */
.flag-ok     { color: #0f7b6c; background: #dbeddb; } /* < 3% */
```

### Overlap Map Design

Simple list format — not a heatmap. Each pair on one line with an overlap score bar:
```
PPFAS Flexi Cap ↔ Mirae Large Cap   ████████░░  62%  High overlap
```

Bar color: red (> 60%), amber (40-60%), green (< 40%).

---

## Error Handling

**Tier 1 active_equity NOT_FOUND:**
Mark fund as `data_unavailable`. Show inline: `[Fund Name] — not indexed in mfdata.in, analysis skipped.`

**Tier 1 index fund NOT_FOUND:**
Fall back to Tier 2. Use embedded approximate index weights. Label clearly as approximate.

**mfdata.in unreachable (network error):**
Show in the tab: `Portfolio X-Ray unavailable — could not reach mfdata.in. Check your internet connection and try again.`
Keep all tiers 2-4 visible — they don't need mfdata.in.

**No MF holdings:**
Show: `No mutual funds in portfolio — X-Ray only applies to mutual fund holdings.`

**Coverage summary:**
Always show: "Stock-level look-through covers ₹X.XL of ₹XX.XL MF portfolio (X%) — remainder attributed at asset class level."

---

## Key Insights to Surface

After building the look-through table, always check and flag:

1. **Top 5 largest effective exposures** — are any surprises? (e.g., a stock the user didn't consciously choose)
2. **Funds with > 60% overlap** — consolidation candidates
3. **Debt exposure via MFs** — many investors don't realise they hold significant debt through hybrid/multi-asset funds
4. **True gold exposure** — across direct ETFs + SGB + gold allocation in multi-asset funds
5. **True international exposure** — US stocks via Nasdaq ETFs + US allocation in Flexi Cap funds like PPFAS

These insights should appear as a short **3-5 line summary at the top of the X-Ray tab** before the data sections.

---

## Example Summary Text

```
Your effective portfolio looks quite different from your surface-level allocation.
HDFC Bank is your single largest position at 6.8% — not from direct equity, but accumulated 
silently across 4 funds. Mirae Large Cap and Axis Bluechip overlap 71% — you're paying for 
two funds doing the same job. True debt exposure via multi-asset funds is 8.4%, nearly double 
what your direct portfolio shows. True gold exposure including fund allocations: 11.2%.
```

Always write this in plain language. One insight per sentence. No bullet points.
