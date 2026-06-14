---
name: calorie-counter
description: >
  Track daily calorie and macro intake for Rajiv and Jasleen using LIVE verified data
  from the FatSecret MCP server at https://rajivfood.duckdns.org/mcp.

  TRIGGER PHRASES — activate this skill when user says ANY of:
  ADD:     "add R", "+ R", "add Rajiv", "log R", "add J", "+ J", "add Jasleen", "log J"
  DATE:    "add R to dd-mm", "add J to dd-mm" (historical logging)
  VIEW:    "calorie R dd-mm", "calorie J dd-mm", "day R", "day J", "summary R", "summary J"
  DELETE:  "delete R [entry_id]", "delete J [entry_id]"
  UPDATE:  "update R [entry_id]", "update J [entry_id]"
  WEIGHT:  "weight R [kg]", "weight J [kg]"
---

# Calorie Counter Skill — FatSecret MCP Edition

## Purpose
Track daily food intake for Rajiv and Jasleen using **live, verified nutritional data**
from the FatSecret global food database via MCP at `https://rajivfood.duckdns.org/mcp`.

> ⚠️ CRITICAL RULES — NEVER BREAK THESE:
> 1. ALWAYS call FatSecret MCP tools — NEVER use Claude's own nutritional estimates
> 2. ALWAYS include the FatSecret food_url link for every food item logged
> 3. ALWAYS show confidence score for every food match
> 4. ALWAYS label data as "✅ FatSecret REST API — Global Database (Live)"
> 5. NEVER guess calories — if FatSecret returns no result, say so explicitly

---

## MCP Server
- **URL:** `https://rajivfood.duckdns.org/mcp`
- **11 Available Tools:**
  - `search_foods` — search single food with confidence score
  - `search_multi_food` — search multiple foods at once, each with confidence score ⭐
  - `get_food` — get full nutrition details + all serving sizes for a food_id
  - `log_food` — log a food entry to diary
  - `get_diary` — get all diary entries for a date (includes entry_ids)
  - `get_day_summary` — full day macro summary with % of target
  - `delete_food_entry` — delete a diary entry by entry_id
  - `update_food_entry` — update quantity/serving/meal of existing entry
  - `search_recipes` — search FatSecret recipe database
  - `log_weight` — log body weight in kg
  - `get_weight_history` — get weight history for date range

---

## Daily Targets
| Person  | Weekday (Mon–Fri) | Weekend (Sat–Sun) |
|---------|-------------------|-------------------|
| Rajiv   | 1,650 kcal        | 2,100 kcal        |
| Jasleen | 1,650 kcal        | 2,100 kcal        |

---

## Step 0 — Validate Person
- Must identify "R/Rajiv" or "J/Jasleen" in the command
- If missing → respond ONLY: **"Please specify — Rajiv or Jasleen?"**
- Maintain SEPARATE tables and data per person

---

## Step 1 — Food Lookup via FatSecret MCP (MANDATORY)

### For single or multi-food entries (PRIMARY FLOW):

```
Step A: Parse user input into individual food names
        e.g. "2 rotis and dal tadka for lunch" → ["roti", "dal tadka"]

Step B: Call FatSecret:search_multi_food
        Input: { foods: ["roti", "dal tadka"], region: "IN" }

Step C: Read confidence scores from response:
        ✅ HIGH (80-100)  → proceed to log automatically
        ⚠️ MEDIUM (50-79) → show match to user, ask "Is this correct?"
        ❌ LOW (0-49)     → show top 3 alternatives, ask user to pick

Step D: For each confirmed HIGH or user-confirmed food:
        Call FatSecret:get_food → Input: { food_id: "[id]" }
        (Gets default serving_id and exact nutrition)

Step E: Call FatSecret:log_food for each food
        Input: {
          food_id:    "[id from search]",
          serving_id: "[default serving_id from get_food]",
          quantity:   [number from user input],
          meal:       "[breakfast/lunch/dinner/other]",
          date:       "[YYYY-MM-DD]"
        }

Step F: Call FatSecret:get_day_summary
        Input: { date: "[YYYY-MM-DD]", calorie_target: 1650 }
        (Gets updated totals for the table)
```

### Confidence Score Explanation (always show this to user):
```
Score = Name Match (0-40) + Position (0-25) + Food Type (0-20) + Context (0-15)
Total = 0 to 100

Name Match:     Does food name contain your search words? Exact = 40pts
Position:       Rank 1 = 25pts, Rank 2 = 18pts, Rank 3 = 12pts
Food Type:      Generic = 20pts (better for Indian foods), Brand = 10pts
Context:        Fewer total results = higher confidence (up to 15pts)
```

---

## Step 2 — Build the Calorie Table

**Header** (bold):
**Rajiv DD-MM Calorie Counter** or **Jasleen DD-MM Calorie Counter**

**Columns:**
| # | Food Name | Confidence | FatSecret Link | Serving | Calories | Protein | Fat | Carbs |
|---|-----------|------------|----------------|---------|----------|---------|-----|-------|

**Rules:**
- Food Name = exactly as returned by FatSecret (not paraphrased)
- Confidence = score/100 + emoji (✅⚠️❌)
- FatSecret Link = clickable food_url from API response
- Source line below table: `✅ All data: FatSecret REST API — Global Database (Live)`

---

## Step 3 — Total Row (second last row)
| **TOTAL** | — | — | — | — | **ΣCal** | **ΣP** | **ΣF** | **ΣC** |

---

## Step 4 — Percentage Row (last row)
- Calories: X / 1650 (weekday) or X / 2100 (weekend) = **XX%**
- Remaining: XXX kcal
- Macro split: Protein XX% | Fat XX% | Carbs XX%

---

## Step 5 — Warning System
- Within 10% of target → no warning
- Exceeded >10% → ⚠️ WARNING: "Exceeded by XXX kcal. Tomorrow's adjusted target: XXXX kcal"

---

## Step 6 — Date Management
- Same table maintained all day for that person
- New table starts next day automatically
- Historical: `add R to DD-MM` → call `get_diary(date)` first then append

---

## Step 7 — Weekly Comparison
After 7 days of tracking → show 7-day comparison table (daily totals vs targets)

---

## Special Commands

### View Diary / Day Summary
```
"day R" or "summary R" or "calorie R today":
→ Call FatSecret:get_day_summary { date: today, calorie_target: 1650 }
→ Show as formatted table

"calorie R DD-MM":
→ Call FatSecret:get_diary { date: YYYY-MM-DD }
→ Show as formatted table
```

### Delete Entry
```
"delete R [entry_id]":
→ Call FatSecret:delete_food_entry { entry_id: "[id]" }
→ Call FatSecret:get_day_summary to refresh totals
→ Show updated table
Note: Get entry_ids by calling "day R" or "calorie R today" first
```

### Update Entry
```
"update R [entry_id] [new_qty] [meal]":
→ Call FatSecret:get_diary to get current serving_id
→ Call FatSecret:update_food_entry { entry_id, serving_id, quantity, meal }
→ Call FatSecret:get_day_summary to refresh
→ Show updated table
```

### Weight Logging
```
"weight R 72.5":
→ Call FatSecret:log_weight { weight_kg: 72.5, date: today }
→ Confirm: "⚖️ Weight 72.5 kg logged for Rajiv on DD-MM"
```

---

## Response Format
- Success: **"Calorie Table updated for [Rajiv/Jasleen] DD-MM"** then show table
- Always show confidence score per food item
- Always show FatSecret URL per food item
- Always show data source line
- No extra commentary beyond the table

---

## Full Example Flow

**User types:** `add R 2 rotis and a bowl of dal tadka lunch`

**Skill does:**
1. Identifies: Rajiv, today, lunch
2. Parses foods: ["roti", "dal tadka"]
3. Calls `search_multi_food(["roti", "dal tadka"], region="IN")`
4. Gets back: roti → 88/100 ✅ HIGH | dal tadka → 72/100 ⚠️ MEDIUM
5. For roti (HIGH): calls `get_food(food_id)` → gets serving_id → calls `log_food`
6. For dal tadka (MEDIUM): shows match → "Did you mean X?" → waits for confirm
7. After confirmation: calls `get_food` → `log_food`
8. Calls `get_day_summary(today, 1650)`
9. Shows table with confidence column + FatSecret links

**Output:**
```
Calorie Table updated for Rajiv 14-06

| # | Food Name     | Confidence  | FatSecret Link          | Serving | Cal | P   | F  | C   |
|---|---------------|-------------|-------------------------|---------|-----|-----|----|-----|
| 1 | Roti/Chapati  | 88/100 ✅   | fatsecret.com/roti...   | 1 roti  | 104 | 3g  | 1g | 22g |
| 2 | Dal Tadka     | 72/100 ⚠️   | fatsecret.com/dal...    | 1 cup   | 198 | 12g | 6g | 28g |
|   | **TOTAL**     |             |                         |         | 302 | 15g | 7g | 50g |
|   | **% of 1650** |             |                         |         | 18% | P24%| F21%|C55%|

✅ All data: FatSecret REST API — Global Database (Live)
```
