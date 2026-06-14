---
name: calorie-counter
description: >
  Track daily calorie and macro intake for Rajiv and Jasleen using LIVE verified data
  from the FatSecret MCP server at https://rajivfood.duckdns.org/mcp.

  TRIGGER PHRASES — activate this skill when user says ANY of:
  ADD:    "add R", "+ R", "add Rajiv", "log R", "add J", "+ J", "add Jasleen", "log J"
          "add [food]" or "+ [food]" (no R/J → ask who, then continue)
  DATE:   "add R to dd-mm", "add J to dd-mm"
  VIEW:   "calorie R dd-mm", "calorie J dd-mm", "day R", "day J", "summary R", "summary J"
  DELETE: "delete R [entry_id]", "delete J [entry_id]"
  UPDATE: "update R [entry_id]", "update J [entry_id]"
  WEIGHT: "weight R [kg]", "weight J [kg]"
---

# Calorie Counter Skill — FatSecret MCP Edition v4

## Purpose
Track daily food intake for Rajiv and Jasleen using **live, verified nutritional data**
from the FatSecret global food database via MCP at `https://rajivfood.duckdns.org/mcp`.

> ⚠️ CRITICAL RULES — NEVER BREAK THESE:
> 1. ALWAYS call FatSecret MCP tools — NEVER use Claude's own nutritional estimates
> 2. ALWAYS include FatSecret food_url link for every food item
> 3. ALWAYS show confidence score for every food match (in table AND as column)
> 4. ALWAYS label data as "✅ FatSecret REST API — Global Database (Live)"
> 5. NEVER guess calories — if FatSecret returns no result, say so explicitly

---

## MCP Server
- **URL:** `https://rajivfood.duckdns.org/mcp`
- **Tools used:** `search_multi_food`, `get_food`, `log_food`, `get_diary`,
  `get_day_summary`, `delete_food_entry`, `update_food_entry`, `search_foods`,
  `search_recipes`, `log_weight`, `get_weight_history`

---

## Daily Targets
| Person  | Weekday (Mon–Fri) | Weekend (Sat–Sun) |
|---------|-------------------|-------------------|
| Rajiv   | 1,650 kcal        | 2,100 kcal        |
| Jasleen | 1,650 kcal        | 2,100 kcal        |

---

## Step 0 — Identify Person

Check if "R", "Rajiv", "J", or "Jasleen" is present in the command.

**If person IS identified:** proceed immediately to Step 1.

**If person is NOT identified:**
- Respond: **"For Rajiv or Jasleen? (R/J)"**
- Remember the entire original food command exactly as typed
- When they reply "R" or "J" → continue with full original command as if they had written R/J from the start
- Do NOT ask them to retype their food entry

---

## Step 1 — Parse the Input

Extract these three things from the user's message:

**A. Person:** R/Rajiv or J/Jasleen

**B. Meal:** breakfast / lunch / dinner / other
- If not specified, infer from time of day or ask once

**C. Food items with quantities** — handle ALL these formats:
```
"add R 100 gm rice"           → rice, 100g
"add R rice 100g"             → rice, 100g
"add R 2 rotis and dal"       → roti ×2, dal ×1
"add J 1 cup poha"            → poha, 1 cup
"add R chicken biryani 250gm" → chicken biryani, 250g
"add R 1 banana and 2 eggs"   → banana ×1, egg ×2
"add J poha"                  → poha, quantity unknown → use default serving
```

Parse each food into: `{ name: "rice", quantity: 100, unit: "gm" }`
If quantity is missing → use FatSecret's default serving size.

---

## Step 2 — Food Lookup via FatSecret MCP (MANDATORY)

```
A. Build food list from parsed items:
   e.g. ["rice", "roti", "dal tadka"]

B. Call FatSecret:search_multi_food
   Input: { foods: ["rice", "roti", "dal tadka"], region: "IN" }

C. Read confidence scores:
   ✅ HIGH (80-100)  → proceed to log automatically
   ⚠️ MEDIUM (50-79) → show match, ask "Is [food name] correct? (y/n)"
   ❌ LOW (0-49)     → show top 3 options, ask user to pick by number

D. For each confirmed food:
   Call FatSecret:get_food { food_id: "[id]" }
   → gets default serving_id and exact per-serving nutrition

E. Scale nutrition to user's quantity:
   If user said 100g and default serving is 30g:
   multiply all macros by (100/30) = 3.33×

F. Call FatSecret:log_food for each confirmed food:
   Input: {
     food_id:    "[id]",
     serving_id: "[default serving_id]",
     quantity:   [scaled quantity],
     meal:       "[meal]",
     date:       "[YYYY-MM-DD]"
   }

G. Call FatSecret:get_day_summary
   Input: { date: "[YYYY-MM-DD]", calorie_target: 1650 }
```

---

## Step 3 — Build the Calorie Table

**Header** (bold, first line):
**Rajiv DD-MM Calorie Counter** or **Jasleen DD-MM Calorie Counter**

**Table columns — in this exact order:**

| # | Food Name | Qty | Confidence | FatSecret Link | Calories | Protein (g) | Fat (g) | Carbs (g) |
|---|-----------|-----|------------|----------------|----------|-------------|---------|-----------|

**Column rules:**
- **#** — row number
- **Food Name** — exactly as returned by FatSecret API
- **Qty** — quantity as user specified (e.g. "100g", "2 rotis", "1 cup")
- **Confidence** — format: `88/100 ✅` or `65/100 ⚠️` or `35/100 ❌`
- **FatSecret Link** — shortened display e.g. `fatsecret.com/rice` linking to full food_url
- **Calories, Protein, Fat, Carbs** — scaled to user's actual quantity

**Why confidence score in the table:**
The confidence column lets Rajiv and Jasleen instantly see at a glance whether
each food was matched exactly or approximately — without needing to click any link.
A score of 88/100 for "rice" means trust it. A score of 45/100 means verify it.
This is especially important for Indian foods where spelling variants are common
(roti/chapati/phulka, dal/dahl/lentils, poha/beaten rice etc.)

---

## Step 4 — Total Row (second last row)

| **TOTAL** | — | — | — | — | **ΣCal** | **ΣP g** | **ΣF g** | **ΣC g** |

---

## Step 5 — Percentage Row (last row)

| **% of target** | — | — | — | — | **XX%** | **P:XX%** | **F:XX%** | **C:XX%** |

- Calories %: (total cal / daily target) × 100
- Remaining: show below table — "**Remaining: XXX kcal**"
- Macro %: protein cals = P×4, fat cals = F×9, carb cals = C×4, each as % of total cal

---

## Step 6 — Warning System

- Within 10% of target → no warning, proceed normally
- Exceeded >10% → add below table:
  **⚠️ WARNING: Exceeded today's target by XXX kcal. Tomorrow's adjusted target: XXXX kcal**

---

## Step 7 — Date Management

- One table per person per day — same table all day, new food appended to bottom
- New day → new table automatically
- Historical entry `add R to DD-MM [food]`:
  → call `get_diary(date)` first to load existing entries
  → append new food, recalculate totals, show full updated table

---

## Step 8 — Weekly Comparison

After 7 days tracked → show comparison table:
| Date | Rajiv Cal | Target | % | Status |
showing all 7 days with ✅ met / ⚠️ over / ❌ under

---

## Special Commands

### View Day
```
"day R" / "summary R" / "calorie R today"
→ get_day_summary(today, 1650) → show full table
```

### View Historical
```
"calorie R 13-06"
→ get_diary(2026-06-13) → show as full table with confidence + links
```

### Delete Entry
```
"delete R [entry_id]"
→ delete_food_entry(entry_id)
→ get_day_summary → show updated table
Note: always call "day R" first to see entry_ids
```

### Update Entry
```
"update R [entry_id] [new_qty] [meal]"
→ get_diary to get serving_id
→ update_food_entry(entry_id, serving_id, quantity, meal)
→ get_day_summary → show updated table
```

### Weight
```
"weight R 72.5"
→ log_weight(72.5, today)
→ confirm: "⚖️ 72.5 kg logged for Rajiv on DD-MM"
```

---

## Response Format

Every successful add response:
1. One line: **"Calorie Table updated for [Rajiv/Jasleen] DD-MM"**
2. The table (with confidence column)
3. **Remaining: XXX kcal**
4. Source line: `✅ FatSecret REST API — Global Database (Live)`
5. Warning only if exceeded >10%
6. Nothing else

---

## Full Example

**User types:** `add R 100 gm rice lunch`

**Skill does:**
1. Person = Rajiv ✅, Meal = lunch ✅, Food = rice 100g ✅
2. Calls `search_multi_food(["rice"], region="IN")`
3. Gets: "White Rice (Cooked)" — 91/100 ✅ HIGH
4. Calls `get_food(food_id)` → default serving = 100g, 130 cal, P:2.7g F:0.3g C:28g
5. Quantity matches default → no scaling needed
6. Calls `log_food(food_id, serving_id, qty=1, meal="lunch")`
7. Calls `get_day_summary(today, 1650)`
8. Shows:

**Rajiv 14-06 Calorie Counter**

| # | Food Name | Qty | Confidence | FatSecret Link | Calories | Protein (g) | Fat (g) | Carbs (g) |
|---|-----------|-----|------------|----------------|----------|-------------|---------|-----------|
| 1 | White Rice (Cooked) | 100g | 91/100 ✅ | fatsecret.com/white-rice | 130 | 2.7 | 0.3 | 28.0 |
| | **TOTAL** | | | | **130** | **2.7** | **0.3** | **28.0** |
| | **% of 1650** | | | | **7.9%** | **P:8%** | **F:2%** | **C:90%** |

**Remaining: 1,520 kcal**
✅ FatSecret REST API — Global Database (Live)
