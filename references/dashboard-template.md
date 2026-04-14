# Campfire Analyst Dashboard — HTML Template Spec

## Overview

The dashboard is a single self-contained `.html` file with **four tabs**, organised by decision urgency:

- **Summary** — headline finding, hero numbers, top 3 quick wins, multibaggers/losers
- **Fix List** — interactive portfolio task center
- **Analysis** — Campfire Vitals, style portrait, top holdings, sector breakdown
- **X-Ray** — mutual fund look-through analysis (opt-in detail)

All four tabs share the same design tokens and feel like one cohesive app. Generated as the very first output the user sees — before investment.md, before the prediction game.

**Output:** `/mnt/user-data/outputs/campfire-dashboard.html`

**CRITICAL: All data must come from THIS user's portfolio. No hardcoded values. Generate this HTML dynamically with the user's actual numbers.**

---

## Design Language

Notion-like minimalist. White background. Inter font. No dark theme. No shadows. No gradients. Color only through small accent elements.

### Tokens

```css
--bg: #ffffff;
--surface: #f7f7f5;
--border: #e8e8e4;
--text: #37352f;
--text-secondary: #787774;
--text-muted: #b4b4b0;
--accent-purple: #6940a5;
--accent-orange: #d9730d;
--accent-red: #e03e3e;
--accent-green: #0f7b6c;
--accent-blue: #2f6beb;
--font: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--mono: 'SF Mono', 'Consolas', 'Fira Code', monospace;
```

### Principles

1. Single page, no scrolling sections — the whole report flows vertically within each tab
2. Sections separated by generous whitespace (40-60px), not heavy borders
3. Numbers in monospace, text in Inter
4. Section headers: 18-20px, semibold, `#37352f`, no uppercase
5. Metric labels: 13px, `#787774`
6. Values: 24-32px for hero numbers, 14-16px for table data
7. Progress bars: 3-4px thin, colored by score range
8. Tables: no outer border, thin `#e8e8e4` row dividers, no alternating row colors
9. Cards: `#f7f7f5` background, 8px border-radius, no border, no shadow
10. Pills/badges: same Notion palette as task center

---

## Tab Navigation

```html
<nav style="border-bottom: 1px solid #e8e8e4; margin-bottom: 40px; display: flex; gap: 24px;">
  <button class="tab active" onclick="switchTab('summary',this)">Summary</button>
  <button class="tab" onclick="switchTab('fixlist',this)">Fix List</button>
  <button class="tab" onclick="switchTab('analysis',this)">Analysis</button>
  <button class="tab" onclick="switchTab('xray',this)">X-Ray</button>
</nav>
```

```css
.tab {
  background: none;
  border: none;
  padding: 8px 0;
  font-size: 14px;
  font-weight: 500;
  color: #787774;
  cursor: pointer;
  border-bottom: 2px solid transparent;
  margin-bottom: -1px;
}
.tab.active {
  color: #37352f;
  border-bottom-color: #37352f;
}
```

Clicking a tab hides the inactive panels and shows the active one. All state in vanilla JS.

---

## Tab 1: Summary Layout

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  [Headline finding — one bold sentence]             │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│  │  ₹X.XXCr │  │  +XX.X%  │  │   XXX    │          │
│  │  wealth   │  │  return  │  │holdings  │          │
│  └──────────┘  └──────────┘  └──────────┘          │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Wealth Composition                                 │
│  ████████████████████░░░░░░  Equity XX%             │
│  ████████████████░░░░░░░░░░  MF XX%                 │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Top 3 Quick Wins                                   │
│                                                     │
│  ⊕  [Task name]          [₹ value]  →              │
│  ✂  [Task name]          [₹ value]  →              │
│  ⚡  [Task name]          [₹ value]  →              │
│                                                     │
│  → See all tasks in Fix List                        │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  🏆 Multibaggers          ⚠️ Biggest Losers          │
│  STOCK_A  +XXX%           STOCK_X  -XX%             │
│  STOCK_B  +XXX%           STOCK_Y  -XX%             │
│  ...top 5 each                                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Headline Finding

Generate one bold sentence using this priority order:

```
1. Effective look-through exposure > 8% on any security
   → "[Security] is your largest real position at X% — held across N funds you didn't consciously choose"

2. High Impact + Quick Win task exists
   → "You have N [asset] vehicles doing the same job — consolidating saves ₹X.XL in complexity"

3. More than 5 multibaggers
   → "Your stock picks are working — N positions have doubled or more, led by [top bagger] at +XXX%"

4. Largest fragmentation issue
   → "[N] gold instruments, [N] ELSS funds — one each is all you need"

5. Return context
   → "Your portfolio is [up XX.X%] overall — [beating / roughly matching / trailing] the Nifty 50 benchmark"
```

Style: `font-size: 18px; font-weight: 600; color: #37352f; line-height: 1.5`. No icon. No background. Just the sentence.

### Top 3 Quick Wins

Pick tasks where `impact === "high"` and `effort === "low"`. Show max 3.
Each row: category icon + task name + ₹ value addressable + arrow.
Clicking a row switches to the Fix List tab and expands that task.

---

## Tab 2: Fix List Layout

Full task center. See `references/task-center-template.md` for complete spec.
Implement in vanilla JS + HTML. Expandable rows, checkbox completion, category filters.

---

## Tab 3: Analysis Layout

```
┌─────────────────────────────────────────────────────┐
│  — Self-assessment ————————————————————————————————│
│  Campfire Vitals              Overall: XX/100       │
│  [8 metrics with bars and one-line notes]           │
├─────────────────────────────────────────────────────┤
│  Investment Style Portrait                          │
│  [150-250 word paragraph]                           │
├─────────────────────────────────────────────────────┤
│  — What am I betting on? ——————————————————————────│
│  Thematic Map                                       │
│                                                     │
│  India Financial Deepening  ████████████  XX%  ₹XL │
│  India PSU Reform           ████████      XX%  ₹XL │
│  India Infrastructure       ███████       XX%  ₹XL │
│  Global Tech (US)           █████         XX%  ₹XL │
│  ...                                               │
│  [2-3 sentence thematic portrait]                  │
│  Gaps: ⚠️ Zero India Defence · Zero Global Consump. │
├─────────────────────────────────────────────────────┤
│  — How am I performing? ————————————————————————— │
│  vs Nifty 50 (approximate)                          │
│  +XX.X% vs Nifty ~XX–XX%  (~X–X yr window)         │
│  Verdict: Beating / Roughly matching / Trailing     │
├─────────────────────────────────────────────────────┤
│  — What are my risks? ——————————————————————————— │
│  Concentration Risk                                 │
│  Direct + MF look-through (as of Mar 2026) ←updates│
│                                                     │
│  Top 1  XX%  [████░░░░░░]  [flag if > 10%]         │
│  Top 3  XX%  [████████░░]  [flag if > 25%]         │
│  Top 5  XX%  [█████████░]                           │
│                                                     │
│  ⚠️ STOCK_A is XX% combined (direct + N funds)      │
│     If it halved → portfolio drops ₹X.XL            │
│  → Full breakdown in Fund Detail tab                │
├─────────────────────────────────────────────────────┤
│  — Portfolio health at a glance ————————————————— │
│  ┌─────────────┐  ┌─────────────┐                  │
│  │ Expense     │  │ Liquidity   │                  │
│  │ ₹X,XXX/yr  │  │ XX% liquid  │                  │
│  │ X.X% of MFs │  │ ₹X.XL locked│                  │
│  └─────────────┘  └─────────────┘                  │
│  ┌─────────────┐  ┌─────────────┐                  │
│  │ Dividend    │  │ Beta        │                  │
│  │ ₹X,XXX/yr  │  │ X.XX        │                  │
│  │ X.X% yield  │  │ [Aggressive]│                  │
│  └─────────────┘  └─────────────┘                  │
├─────────────────────────────────────────────────────┤
│  — The raw data ————————————————————————————————— │
│  Top 10 Holdings  (direct + MF, by value)           │
├─────────────────────────────────────────────────────┤
│  Sector Allocation (direct equity)                  │
│  [horizontal bars]                                  │
└─────────────────────────────────────────────────────┘
```

**On page load:** Concentration Risk shows direct-equity-only numbers with label "Direct equity only — loading MF data...". When background X-Ray fetch completes, numbers update to combined view.

---

## Tab 4: X-Ray Layout

The detail view behind the Concentration Risk numbers. Shows the supporting evidence — not a second concentration view.

Contents:
1. Orientation note: "These are the fund holdings behind your Concentration Risk numbers"
2. Full security-by-security look-through table (sortable, filterable)
3. Fund overlap map — which fund pairs are redundant
4. Asset class look-through — true allocation across equity/debt/gold/international/cash
5. Coverage summary — which funds loaded live, which are approximate, which are skipped

Data loads in background on page open. Tab shows a loading indicator until ready.
See `references/xray-template.md` for full spec.

---

## Implementation Notes

### HTML Structure

Single self-contained HTML file. Inline CSS. No external dependencies except Google Fonts (Inter).

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Campfire Analyst — Portfolio Report</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    /* All styles inline — Notion-like tokens */
  </style>
</head>
<body>
  <!-- Sections as described above -->
</body>
</html>
```

### Section Styling

```css
body {
  font-family: 'Inter', -apple-system, sans-serif;
  background: #ffffff;
  color: #37352f;
  max-width: 720px;
  margin: 0 auto;
  padding: 40px 24px;
  line-height: 1.6;
}

.section {
  margin-bottom: 48px;
}

.section-title {
  font-size: 18px;
  font-weight: 600;
  color: #37352f;
  margin-bottom: 16px;
}

.hero-number {
  font-family: 'SF Mono', 'Consolas', monospace;
  font-size: 28px;
  font-weight: 700;
  color: #37352f;
}

.label {
  font-size: 13px;
  color: #787774;
}

.card {
  background: #f7f7f5;
  border-radius: 8px;
  padding: 16px 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

td, th {
  padding: 8px 0;
  border-bottom: 1px solid #e8e8e4;
  font-size: 14px;
}

th {
  text-align: left;
  font-weight: 500;
  color: #787774;
  font-size: 12px;
}
```

### Campfire Vitals Score Bars

```css
.vitals-bar {
  height: 4px;
  background: #e8e8e4;
  border-radius: 2px;
  width: 120px;
}

.vitals-fill {
  height: 100%;
  border-radius: 2px;
  /* Color by score: green (#0f7b6c) > 75, amber (#d9730d) 50-75, red (#e03e3e) < 50 */
}
```

### Sector Bars

Horizontal bars using simple div widths. Max width = largest sector %. Color: `#37352f` at 30% opacity for a soft, Notion-like feel. No bright colors for sector bars — let the numbers speak.

### Responsive

- Max-width 720px, centered
- Hero numbers stack vertically on mobile
- Tables become scrollable on narrow screens
- Padding reduces on mobile (24px → 16px)

---

## Generation Instructions for the Skill

When generating this dashboard, the skill should:

1. Compute all values from the raw holdings/MF data
2. Calculate all 8 Campfire Vitals scores using the vitals-metrics.md reference
3. Write the investment style portrait paragraph
4. Compute thematic map (section 1.9) — assign every holding to themes, compute ₹ and % per theme, write thematic portrait
5. Compute concentration risk (section 1.7) — top 1/3/5/10%, sector flags, halving scenario
6. Compute benchmark context (section 1.8) — infer portfolio age, compute verdict
7. Compute expense ratio audit (section 1.10) — annual cost in ₹, savings opportunity
8. Compute liquidity profile (section 1.11) — % accessible quickly, locked value
9. Compute dividend yield (section 1.12) — estimated annual income
10. Compute weighted portfolio beta (section 1.13)
11. Generate all portfolio tasks using task-center-template.md — including thematic gap, expense, and liquidity tasks
12. Classify all MF funds into tiers and prepare the JavaScript X-Ray engine using xray-template.md
13. Pick the single headline finding using the priority order in the Summary tab spec
14. Identify the top 3 Quick Win tasks (high impact + low effort)
15. Select top 10 holdings across both direct + MF by current value
16. Select top 5 multibaggers and top 5 losers
17. Build the sector allocation from direct equity
18. Inject all values into a single HTML file with four tabs: Summary, Fix List, Analysis, X-Ray
19. Save to `/mnt/user-data/outputs/campfire-dashboard.html`
20. Present to user

**The dashboard is the FIRST visual output the user sees.** Summary tab gives the headline and the three most actionable wins immediately. Fix List tab is the full task center. Analysis tab goes deeper. X-Ray tab reveals what's actually inside every mutual fund.
