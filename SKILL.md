---
name: campfire-analyst
description: "Your portfolio is a fingerprint. Campfire Analyst reads it. Connects to Zerodha, runs a full-stack analysis — sector breakdown, Campfire Vitals scoring, fragmentation audit, actionable task list — and then plays a prediction game: your phone, your car, your hotel tier, all inferred from your holdings. Three output files: campfire-dashboard.html, investment.md, taste.md. Trigger on: portfolio review, Campfire Vitals report, investment.md, taste.md, predict me from stocks, campfire analyst, task center, consumer identity, financial health check."
author: Subodh Kolhe
license: Apache 2.0 — complete terms in LICENSE.txt
---

# Campfire Analyst

---

## For Users: How to Use This Skill

### What it does
Campfire Analyst reads your Zerodha portfolio and produces a complete picture of your investments — what you own, how it's performing, what's broken, and what your money reveals about you as a person.

It generates three files:
- **campfire-dashboard.html** — your portfolio at a glance, plus a full task list of things to fix
- **investment.md** — a detailed wealth document you can save or share with an advisor
- **taste.md** — a lifestyle identity profile built through a prediction game

### How to start
Just type `/campfire-analyst` in the chat. That's it. No setup, no options to configure.

You'll be asked to log into your Zerodha account via a one-click link. Once authorized, the analysis runs automatically.

### What happens next
1. You'll see the **Campfire Dashboard** — a two-tab HTML file. The Overview tab shows your portfolio health, Campfire Vitals scores, top holdings, and sector breakdown. The Task Center tab lists everything worth fixing, ranked by impact.
2. **investment.md** follows immediately — the full wealth document with deep allocation tables.
3. Then the **prediction game** begins — Claude tries to guess your phone, car, watch, hotel preference and more from your portfolio alone, one prediction at a time. At the end it generates taste.md.

### If you only want part of it
You can ask for specific pieces directly:
- *"Just give me my Campfire Vitals report"*
- *"Generate my investment.md"*
- *"Let's do the prediction game"*
- *"Show me my task center"*

### If you don't use Zerodha
You can still run the analysis manually. Share your holdings as a screenshot, CSV, or just paste your top 20-30 positions with quantities and average buy prices. Claude will work with whatever you provide.

### What it doesn't do
It doesn't execute any trades, place orders, or modify your account in any way. It only reads your holdings data.

---

## For Skill Authors: Framework Reference

A six-part portfolio intelligence framework that transforms raw holdings data into a complete financial + consumer identity profile.

## Overview

When triggered, this skill runs **six sequential analyses** on the user's portfolio:

| # | Analysis | What It Produces | Reference File |
|---|----------|-----------------|----------------|
| 1 | Deep Portfolio Analysis | Sector breakdown, winners/losers, asset class mix, MF breakdown | (computed internally — feeds dashboard) |
| 2 | Campfire Vitals Report | 8 custom financial health markers scored 0-100 | `references/vitals-metrics.md` (computed internally — feeds dashboard) |
| 3 | Investment Style Portrait | Written paragraph characterising the investor's style and blind spots | (computed internally — feeds dashboard) |
| 4 | Campfire Dashboard + Task Center | Tabbed HTML file — **Overview tab** (Parts 1-3 visual summary) + **Task Center tab** (actionable fix list) | `references/dashboard-template.md`, `references/task-center-template.md` |
| 5 | investment.md Generation | Complete wealth document with deeper consolidated allocation tables | (file output) |
| 6 | taste.md Consumer Identity | Archetype extraction → lifestyle prediction game | `references/archetypes.md`, `references/prediction-chains.md` |

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

> **Do not present this section inline.** Compute all values silently. Everything here feeds directly into the Campfire Dashboard (Part 4), which is the first and only time this data is shown to the user.

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
- **Fragmentation Report** — identify where the same asset class is spread across multiple vehicles (e.g., 5 Nifty 50 funds, 4 gold ETFs). Flag any asset tracked by 3+ vehicles.

### 1.6 Consolidated Views

Build cross-cutting views that merge direct equity and MF data:
- **Precious Metals (all vehicles)** — every gold/silver instrument across direct + MF
- **International Exposure (all vehicles)** — every US/global instrument
- **Nifty 50 Trackers (all vehicles)** — every Nifty/Sensex instrument

These consolidated views reveal the TRUE allocation that individual portfolio views hide.

---

## Part 2: Campfire Vitals Report

> **Do not present this section inline.** Compute all 8 scores silently. Output flows exclusively into the Campfire Dashboard (Part 4).

Read the reference file for metric definitions:
→ See `references/vitals-metrics.md`

Score each of the 8 financial health markers on a 0-100 scale. Map them from biological readiness metrics to financial equivalents. Present as a dashboard with:
- Each metric name, score, and one-line interpretation
- Overall Financial Recovery Score (weighted average)
- One paragraph summary of financial fitness

**The Campfire Vitals metaphor:** Just as The Campfire Vitals framework maps biological readiness metrics through HRV, RHR, sleep quality etc., Campfire Vitals tracks portfolio readiness through diversification quality, risk calibration, capital efficiency etc.

---

## Part 3: Investment Style Portrait

> **Do not present this section inline.** Write the paragraph internally. It is presented once, inside the Campfire Dashboard (Part 4).

Write a single rich paragraph (150-250 words) that captures:

1. **Primary investing style** — what kind of investor is this person? (Not a label — a description of HOW they invest based on observable behavior)
2. **Strengths** — what the portfolio reveals they're good at
3. **Blind spots** — what structural gaps or biases the data reveals
4. **The contradiction** — almost every portfolio has one (e.g., "researches stocks deeply but fragments gold across 7 vehicles without consolidating")
5. **The signature move** — their most distinctive pattern (e.g., "accumulates winners quietly" or "IPO hunter who holds through drawdowns")

**CRITICAL: Write this from the data, not from assumptions. Read the actual holdings, the actual returns, the actual allocation. Two portfolios with the same total value can reveal completely different personalities.**

---

## Part 4: Campfire Dashboard + Task Center

**This is the first thing the user sees — and the only place Parts 1-3 are presented.** Do not summarise or preview any analysis before generating this. Go straight from data-pulling to this file.

Generate a **single self-contained HTML file** with two tabs at the top:

- **Overview tab** — the full visual portfolio summary (Parts 1-3)
- **Task Center tab** — the interactive fix list (what was previously Part 7)

Read both reference files for the full spec:
→ See `references/dashboard-template.md` for the Overview tab design
→ See `references/task-center-template.md` for the Task Center tab design

### Tab Structure

```html
<!-- Two tabs, clean pill or underline style -->
<nav>
  <button class="tab active">Overview</button>
  <button class="tab">Task Center</button>
</nav>
```

- Tabs are top-level navigation — toggling between them hides/shows each panel
- Active tab uses a subtle underline or pill indicator in `#37352f`
- Inactive tab text in `#787774`
- No heavy tab borders or backgrounds — keep it Notion-like

### Overview Tab Contents

1. **Hero metrics** — total wealth, overall return %, total holdings count (3 large numbers at top)
2. **Wealth composition** — horizontal stacked bar showing direct equity vs. MF vs. cash split
3. **Campfire Vitals Report** — all 8 scores with thin colored progress bars and one-line interpretations
4. **Investment Style Portrait** — the paragraph from Part 3
5. **Top 10 Holdings** — merged list across direct equity and MF, sorted by current value
6. **Sector Allocation** — horizontal bars for direct equity sectors
7. **Key Findings** — 4-6 of the most important structural issues, with category icons (⊕ ⚖ ✂ ⚡ ◎). This is a **teaser, not a full list** — each finding should be one tight line. End with "→ See Task Center for full fix list" with a clickable link that switches to the Task Center tab.
8. **Multibaggers & Losers** — top 5 each, side by side

### Task Center Tab Contents

Generate all portfolio fix tasks as per `references/task-center-template.md`. The task data, categories, interaction model, and design spec are unchanged — it just lives inside this HTML file instead of a separate JSX artifact.

Implement the task center in **vanilla JS + HTML** (not React), consistent with the HTML file format. Use `<details>`/`<summary>` or click handlers for expand/collapse. All state in JS variables.

### Shared Design

Both tabs share the same design tokens — same font, colors, spacing. The file feels like one cohesive app, not two things stitched together.

```css
/* Shared tokens — same as dashboard-template.md */
--bg: #ffffff;
--surface: #f7f7f5;
--border: #e8e8e4;
--text: #37352f;
--text-secondary: #787774;
```

**Design:** Notion-like minimalist. White background, `#f7f7f5` card surfaces, Inter font, thin `#e8e8e4` dividers. No dark theme, no shadows, no gradients. Max-width 720px, centered. Self-contained HTML with inline CSS and inline JS.

Save to `/mnt/user-data/outputs/campfire-dashboard.html` and present to user.

---

## Part 5: investment.md Generation

## Part 5: investment.md Generation

Compile Parts 1-3 into a structured markdown document with these sections:

1. Combined Wealth Overview (the master table)
2. Asset Allocation (full portfolio, merged direct + MF)
3. Direct Equity Portfolio (snapshot, sectors, top positions, baggers, losers)
4. Mutual Fund Portfolio (overview, categories, top holdings, AMC concentration)
5. Consolidated Views (precious metals, international, Nifty trackers — with fragmentation notes)
6. Investor Identity Profile (style tags, behavioral signals, consumer personality type)
7. Portfolio Gaps & Opportunities (table of gaps with current state + suggestion)

Save to `/mnt/user-data/outputs/investment.md` and present to user.

**DO NOT include any personal details (name, city, property, phone model, car, etc.) in investment.md. It should contain ONLY what the portfolio data itself reveals. Personal details belong in taste.md after the prediction game validates them.**

---

## Part 6: taste.md Consumer Identity

**This is the interactive, conversational part. It should feel like a game, not a report.**

### 5.1 Extract the Personality Archetype

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

### 5.2 The Prediction Game

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

### 5.3 Generate taste.md

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
2. **Compute silently** (Parts 1-3) — run all analysis, score Campfire Vitals metrics, write style portrait, generate task list. **Do not output anything to the user yet.**
3. **Campfire Dashboard + Task Center** (Part 4) — generate single tabbed HTML file, save, present. **This is the first thing the user sees.**
4. **Generate investment.md** (Part 5) — save the full wealth document, present to user
5. **taste.md Prediction Game** (Part 6) — interactive, takes multiple turns

Steps 1-4 run in a single response. Part 5 follows immediately after. Part 6 requires back-and-forth conversation.

If the user asks for a specific part only (e.g., "just give me my Campfire Vitals report" or "generate my taste.md"), jump to that section directly — but always pull data first.

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
- **campfire-dashboard.html** = the consolidated app (Overview tab + Task Center tab — first thing the user sees)
- **investment.md** = the wealth document (pure data, no personal predictions)
- **taste.md** = the identity document (predictions, validated through conversation)

The dashboard gives the overview and the fix list in one place. investment.md goes deep. taste.md gets personal.
