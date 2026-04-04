# Campfire Analyst Dashboard — HTML Template Spec

## Overview

The dashboard is a single self-contained `.html` file with **two tabs**:

- **Overview tab** — visual summary of the entire campfire-analyst run (Parts 1-3)
- **Task Center tab** — interactive portfolio fix list (previously a separate artifact)

Both tabs share the same design tokens and feel like one cohesive app. Generated as the very first output the user sees — before investment.md, before the prediction game.

**Output:** `/mnt/user-data/outputs/campfire-dashboard.html`

**CRITICAL: All data must come from THIS user's portfolio. No hardcoded values. The skill should generate this HTML dynamically with the user's actual numbers.**

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

1. Single page, no scrolling sections — the whole report flows vertically
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
  <button class="tab active">Overview</button>
  <button class="tab">Task Center</button>
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

Clicking a tab hides the inactive panel and shows the active one. All state in vanilla JS.

---

## Dashboard Layout

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  Campfire Analyst                                   │
│  Portfolio Intelligence Report                      │
│  Generated Mar 3, 2026 · Zerodha Kite              │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│  │  ₹1.94Cr │  │  +34.4%  │  │   149    │          │
│  │  total    │  │  return  │  │holdings  │          │
│  └──────────┘  └──────────┘  └──────────┘          │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Wealth Composition                                 │
│                                                     │
│  ┌─────────────────────────────────────────┐        │
│  │  ██████████████████░░░░░░░░░░  MF 55%  │        │
│  │  ████████████████░░░░░░░░░░░░  Eq 44%  │        │
│  │  ░░░░░░░░░░░░░░░░░░░░░░░░░░░  Cash 1% │        │
│  └─────────────────────────────────────────┘        │
│                                                     │
│  Direct Equity    ₹84.9L   +45.9%                   │
│  Mutual Funds     ₹1.07Cr  +26.4%                   │
│  Cash             ₹2.4L                              │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Campfire Vitals Report              Overall: 71    │
│                                                     │
│  Diversification    78  ████████░░  Good spread     │
│  Cost Efficiency    58  ██████░░░░  Fragmented      │
│  Risk Calibration   82  ████████░░  Well-matched    │
│  Growth Momentum    91  █████████░  Strong gains    │
│  Rebalancing        48  █████░░░░░  Needs cleanup   │
│  Income Streams     72  ███████░░░  Decent yield    │
│  Regime Adapt.      74  ███████░░░  Gold helps      │
│  Capital Effic.     65  ██████░░░░  Excess buffer   │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Investment Style Portrait                          │
│                                                     │
│  "A dual-strategy investor running active           │
│   direct equity alongside passive MF indexing..."   │
│  (150-250 word paragraph)                           │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Top Holdings                                       │
│                                                     │
│  SBIN          ₹9.25L   +119.7%                    │
│  BHARTIARTL    ₹9.14L   +57.5%                     │
│  PP Flexi Cap  ₹22.1L   +20.3%                     │
│  ...top 10 across both direct + MF                  │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Sector Allocation (direct equity)                  │
│                                                     │
│  Banking & Finance  ████████████  24.4%             │
│  Telecom            █████████     11.1%             │
│  Gold/Silver        ████████      10.7%             │
│  ...                                                │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Key Findings                                       │
│                                                     │
│  ⊕  8 Nifty vehicles tracking same index            │
│  ⊕  7 gold instruments across direct + MF           │
│  ⚖  International at 4% (target: 10-15%)           │
│  ⚖  PPFAS = 33% of MF portfolio                    │
│  ✂  15+ micro-positions under ₹10K                 │
│  ⚡  ₹12.4L in low-yield liquid/arbitrage           │
│                                                     │
│  → Full task list in Portfolio Task Center           │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Multibaggers (25)           Biggest Losers         │
│                                                     │
│  NATIONALUM  +451%           ITCHOTELS  -62%        │
│  LTFOODS     +290%           RBA        -59%        │
│  TMCV        +273%           RAYMONDREL -56%        │
│  ...top 5 each                                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

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

### Key Findings

Each finding gets a category icon (⊕ ⚖ ✂ ⚡ ◎) colored with the Notion category palette. Presented as a simple list with generous line-height, not a table.

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
4. Generate all portfolio tasks using the task-center-template.md reference
5. Identify key findings (4-6 lines, teaser only — full list is in the Task Center tab)
6. Select top 10 holdings across both direct + MF by current value
7. Select top 5 multibaggers and top 5 losers
8. Build the sector allocation from direct equity
9. Inject all values into a single HTML file with two tabs: Overview and Task Center
10. Save to `/mnt/user-data/outputs/campfire-dashboard.html`
11. Present to user

**The dashboard is the FIRST visual output the user sees.** Overview tab gives the complete picture at a glance. Task Center tab is immediately accessible without opening a second file.
