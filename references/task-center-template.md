# Portfolio Task Center — Template & Design Spec

## Overview

The task center is the **second tab of `campfire-dashboard.html`** — it lives inside the same file as the Overview tab, not as a separate React artifact. It presents portfolio fix tasks in an interactive checklist, implemented in **vanilla JS + HTML** consistent with the rest of the dashboard file.

**CRITICAL: Tasks must be derived from THIS user's actual portfolio data. Do not use a static task list. Analyze the portfolio, identify issues, and generate tasks specific to what the data reveals.**

---

## Task Generation Rules

### How to Identify Tasks

Scan the completed analysis for these patterns:

**Consolidation Tasks** (category: "consolidation")
- Any asset class tracked by 3+ vehicles → generate a "Consolidate X" task
- Common: Nifty 50/Sensex across multiple MFs + ETFs, gold across multiple ETFs + SGBs + MFs
- Include: current vehicle count, total value, recommended target (1-2 vehicles), which to keep and which to exit

**Rebalance Tasks** (category: "allocation")
- International exposure < 8% of total → generate "Increase International" task
- Single sector > 30% of direct equity → generate "Review X Concentration" task
- Single AMC > 30% of MF portfolio → generate "Review AMC Concentration" task
- No debt allocation and portfolio > ₹50L → generate "Add Debt Allocation" task
- Small/mid cap < 10% of MF portfolio → consider generating rebalance task

**Cleanup Tasks** (category: "cleanup")
- Positions with > -40% return AND small absolute size (< ₹25K) → generate "Exit Deep Losers" task
- Count positions under ₹10K each; if 10+ → generate "Trim Micro-Positions" task
- Demerger artifacts held with no thesis → flag for review

**Optimization Tasks** (category: "optimization")
- Cash + liquid + arbitrage > 10% of portfolio → generate "Deploy Excess Buffer" task
- Multiple ELSS funds → generate "Optimize ELSS" task
- Regular plan MFs instead of direct plan → generate "Switch to Direct Plans" task
- SIP spread too thin across many funds → generate "Consolidate SIPs" task

**Tracking Tasks** (category: "tracking")
- No clear XIRR tracking → generate "Set Up XIRR Tracking" task
- No benchmark comparison visible → generate "Compare vs. Benchmark" task
- No rebalancing cadence → generate "Set Rebalancing Schedule" task

### Task Fields

Each task object needs:

```javascript
{
  id: Number,              // Sequential
  category: String,        // "consolidation" | "allocation" | "cleanup" | "optimization" | "tracking"
  title: String,           // The fix in 5-8 words
  subtitle: String,        // The metric (e.g., "8 vehicles → 2 vehicles")
  description: String,     // 2-3 sentences explaining what the data shows
  impact: String,          // "high" | "medium" | "low"
  effort: String,          // "low" (quick win) | "medium" (some effort) | "high" (project)
  value: String,           // ₹ amount affected (e.g., "₹14.7L across 8 vehicles")
  action: String,          // Concrete next steps (2-4 sentences)
  status: "todo",          // Always starts as todo
  tags: String[],          // 2-4 tags like ["MF", "ETF", "Simplify"]
}
```

### Impact/Effort Guidelines

**High Impact:** Affects >10% of portfolio, or fixes a structural risk, or unlocks significant alpha
**Medium Impact:** Affects 3-10% of portfolio, or improves efficiency meaningfully
**Low Impact:** Affects <3% of portfolio, or is cosmetic/organizational

**Low Effort (Quick Win):** Single action, no tax implications, can be done in one sitting
**Medium Effort:** Multiple steps, some tax considerations, requires comparison/research
**High Effort (Project):** Ongoing process, complex tax implications, requires multiple sessions

---

## React Dashboard Design Spec

### Visual Design

**Theme:** Notion-like light minimalist. Clean, airy, typographic. No dark backgrounds. No heavy borders. Feels like a well-organized personal workspace.

**Color Palette:**
- Background: `#ffffff`
- Surface/hover: `#f7f7f5` (Notion's warm gray)
- Border: `#e8e8e4` (barely visible dividers)
- Text primary: `#37352f` (Notion's warm black)
- Text secondary: `#787774` (Notion's secondary)
- Text muted: `#b4b4b0`

**Fonts:**
- All text: `'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`
- Monospace accents (values, numbers): `'SF Mono', 'Consolas', monospace`
- Headings: Same as body but heavier weight. No separate display font.

**Category Colors (muted, Notion-style):**
```javascript
const CATEGORIES = {
  consolidation: { label: "Consolidate", color: "#6940a5", bg: "#f3e8ff", icon: "⊕" },  // purple
  allocation:    { label: "Rebalance",   color: "#d9730d", bg: "#fdecc8", icon: "⚖" },  // orange
  cleanup:       { label: "Clean Up",    color: "#e03e3e", bg: "#fbe4e4", icon: "✂" },  // red
  optimization:  { label: "Optimize",    color: "#0f7b6c", bg: "#dbeddb", icon: "⚡" },  // green
  tracking:      { label: "Track",       color: "#2f6beb", bg: "#d3e5ef", icon: "◎" },  // blue
};
```

**Impact Badges (soft pills):**
```javascript
const IMPACT_CONFIG = {
  high:   { label: "High Impact", color: "#e03e3e", bg: "#fbe4e4" },
  medium: { label: "Medium",      color: "#d9730d", bg: "#fdecc8" },
  low:    { label: "Low",         color: "#787774", bg: "#f1f1ef" },
};
```

**Effort Badges (soft pills):**
```javascript
const EFFORT_CONFIG = {
  low:    { label: "Quick Win",   color: "#0f7b6c", bg: "#dbeddb" },
  medium: { label: "Some Effort", color: "#d9730d", bg: "#fdecc8" },
  high:   { label: "Project",     color: "#e03e3e", bg: "#fbe4e4" },
};
```

### Design Principles (Notion-like)

1. **Whitespace is the design.** Generous padding, breathing room between elements. Nothing cramped.
2. **Borders are almost invisible.** Use `#e8e8e4` dividers sparingly — prefer spacing over lines.
3. **No heavy UI chrome.** No shadows, no gradients, no rounded-corner cards with borders. Flat and clean.
4. **Color through text and pills only.** Background stays white/warm-gray. Color comes from small accent pills and category labels.
5. **Checkboxes should feel hand-drawn.** Simple square outline, subtle rounded corners (2px), thin border. When checked: filled with the category color, white checkmark.
6. **Hover states are subtle.** Background shifts to `#f7f7f5` on hover. No color explosions.
7. **Typography does the heavy lifting.** Clear hierarchy through size and weight alone. No uppercase labels (except tiny category tags). No letter-spacing tricks.
8. **Icons are emoji, not icon libraries.** Keep the ⊕ ⚖ ✂ ⚡ ◎ category icons. They feel native and Notion-esque.

### Layout Structure

```
┌─────────────────────────────────────────────┐
│ Portfolio Task Center                       │
│ 10 things to fix · ₹X.XCr total wealth     │
│                                             │
│ ████████░░ 2 of 10 complete                 │
│                                             │
│ All  ⊕ Consolidate  ⚖ Rebalance  ✂ Clean  │
│                                             │
├─────────────────────────────────────────────┤
│ ☐  ⊕ Consolidate · High Impact · Quick Win │
│    Consolidate Nifty 50 Trackers            │
│    8 vehicles → 2 vehicles         ₹14.7L  │
│                                             │
│    (expanded detail area, warm gray bg)     │
│    Description text...                      │
│                                             │
│    Next steps                               │
│    Action text...                           │
│                                             │
│    MF · ETF · Simplify                      │
│                                             │
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤
│ ☐  Next task...                             │
└─────────────────────────────────────────────┘
```

Key layout notes:
- No card borders. Tasks separated by thin `#e8e8e4` bottom borders or just whitespace.
- Expanded detail area uses `#f7f7f5` background — barely different from white.
- Tags are small inline pills with colored background, not bordered chips.
- Progress bar is thin (3-4px), category-colored, with warm gray track.
- Header has no background color — just bold text on white.

### Interaction

- **Click checkbox** → toggles done/todo, completed tasks fade to 40% opacity with strikethrough title
- **Click task row** → expands/collapses detail panel (warm gray `#f7f7f5` background, no border)
- **Category pills** → filter by category (active pill gets colored background from category bg)
- **Sort buttons** → sort by impact, effort, or category. Active sort has subtle underline, not heavy background.
- **Status filter** → show all / done only / todo only
- **Progress bar** → thin bar under header tracking completion. Turns green (`#0f7b6c`) when all done.

### Footer

Minimal sticky footer at bottom:
- Left: quick wins count + total addressable capital
- Right: date stamp
- Font size small, muted color, no heavy border — just a thin top divider

### Technical Notes

- Implemented in **vanilla JS + HTML** — no React, no external libraries
- Use `addEventListener('click', ...)` for checkbox toggles, row expand/collapse, tab filters
- Store task state in a plain JS array; re-render relevant DOM nodes on interaction
- Use `<details>`/`<summary>` or manual click toggling for expand/collapse
- All styles inline — consistent with the dashboard's Notion-like design tokens
- Responsive — should work on mobile widths
- **No box shadows, no gradients, no rounded card borders.** The Notion aesthetic is flat, typographic, and relies on whitespace.

---

## Example: Generating Tasks from a Portfolio

Given a portfolio with:
- 6 Nifty 50 MFs + 2 Nifty ETFs = 8 Nifty vehicles
- 4 gold ETFs + 1 SGB + 1 silver MF = 6 precious metals vehicles  
- International at 4% of total
- 15 positions under ₹10K
- Cash + liquid at 12% of portfolio
- 3 ELSS funds
- One AMC at 33% of MF portfolio
- 5 positions down 40%+

This would generate approximately 8-12 tasks:
1. Consolidate Nifty Trackers (consolidation, high, medium)
2. Consolidate Gold Vehicles (consolidation, medium, medium)
3. Increase International (allocation, high, low)
4. Review AMC Concentration (allocation, medium, medium)
5. Add Debt Allocation (allocation, medium, low)
6. Exit Deep Losers (cleanup, medium, low)
7. Trim Micro-Positions (cleanup, low, low)
8. Deploy Excess Buffer (optimization, medium, low)
9. Optimize ELSS (optimization, low, low)
10. Set Up XIRR Tracking (tracking, low, medium)

**Every portfolio will generate a DIFFERENT task list.** A clean, concentrated portfolio might only have 3-4 tasks. A sprawling, fragmented one might have 15+. The task count itself is informative.
