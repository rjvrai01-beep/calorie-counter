---
name: calorie-counter
description: >
  Track daily calorie and macro intake for Rajiv and Jasleen using LIVE verified data from 
  the FatSecret MCP server at https://rajivfood.duckdns.org/mcp.
  
  TRIGGER PHRASES — activate this skill when the user says:
  For Rajiv:   "add R", "+ R", "add Rajiv", "+ Rajiv", "log R", "log Rajiv"
  For Jasleen: "add J", "+ J", "add Jasleen", "+ Jasleen", "log J", "log Jasleen"
  For either:  "add R to dd-mm", "add J to dd-mm" (historical date logging)
  For view:    "calorie R dd-mm", "calorie J dd-mm", "calorie Rajiv dd-mm", "calorie Jasleen dd-mm"
  For delete:  "delete R [entry_id]", "delete J [entry_id]"
  For update:  "update R [entry_id]", "update J [entry_id]"
  For summary: "summary R", "summary J", "day R", "day J"
  For weight:  "weight R [kg]", "weight J [kg]"
---

# Calorie Counter Skill — FatSecret MCP Edition

## Purpose
Track daily food intake for Rajiv and Jasleen using **live, verified nutritional data** from the 
FatSecret global food database via the MCP server at `https://rajivfood.duckdns.org/mcp`.

> ⚠️ **CRITICAL:** This skill MUST always call FatSecret MCP tools for nutritional data.
> NEVER use Claude's internal knowledge or estimates for calories/macros.
> All data must come from the FatSecret API and include verification links.

---

## MCP Server Details
- **Endpoint:** `https://rajivfood.duckdns.org/mcp`
- **Available Tools:**
  - `FatSecret:search_foods` — Search food database
  - `FatSecret:get_food` — Get detailed nutrition + serving IDs
  - `FatSecret:log_food_natural` — NLP natural language logging (Premier)
  - `FatSecret:log_food` — Manual food logging
  - `FatSecret:get_diary` — Get diary entries with entry_ids
  - `FatSecret:get_day_summary` — Full day macro summary
  - `FatSecret:delete_food_entry` — Delete a diary entry
  - `FatSecret:update_food_entry` — Update a diary entry
  - `FatSecret:get_recent_foods` — Recently eaten foods
  - `FatSecret:search_recipes` — Search recipes
  - `FatSecret:log_weight` — Log body weight
  - `FatSecret:get_weight_history` — Weight history

---

## Daily Calorie Targets
| Person  | Weekday (Mon–Fri) | Weekend (Sat–Sun) |
|---------|-------------------|-------------------|
| Rajiv   | 1,650 kcal        | 2,100 kcal        |
| Jasleen | 1,650 kcal        | 2,100 kcal        |

---

## When to Use This Skill

### Adding Food (Today)
- `add R [food] [quantity]` or `+ R [food] [quantity]`
- `add J [food] [quantity]` or `+ J [food] [quantity]`
- `add Rajiv [food] [quantity]` / `add Jasleen [food] [quantity]`
- Example: `add R 2 rotis and dal tadka lunch`

### Adding Food (Historical Date)
- `add R to dd-mm [food] [quantity]`
- `add J to dd-mm [food] [quantity]`
- Example: `add R to 13-06 poha breakfast`

### Viewing Diary
- `calorie R dd-mm` or `calorie Rajiv dd-mm`
- `calorie J dd-mm` or `calorie Jasleen dd-mm`
- `day R` or `summary R` — today's full summary

### Deleting an Entry
- `delete R [entry_id]` — delete Rajiv's entry
- `delete J [entry_id]` — delete Jasleen's entry
- Entry IDs are shown when you view the diary

### Updating an Entry
- `update R [entry_id] [new_quantity] [meal]`
- `update J [entry_id] [new_quantity] [meal]`

### Weight Logging
- `weight R 72.5` — log Rajiv's weight as 72.5 kg
- `weight J 58.0` — log Jasleen's weight

---

## Step-by-Step Instructions

### Step 0: Validate User
- Check if command includes "R", "Rajiv", "J", or "Jasleen"
- If NOT specified, respond ONLY with: **"Please specify — Rajiv or Jasleen?"**
- Maintain SEPARATE tables and diary data per person

---

### Step 1: Look Up Food via FatSecret MCP (MANDATORY)

**ALWAYS use this exact tool-call chain — do not skip:**

#### Option A — Natural Language (Preferred if NLP scope available)
```
Call: FatSecret:log_food_natural
Input: { 
  description: "[exact food and quantity from user]",
  meal: "[breakfast/lunch/dinner/other]",
  date: "[YYYY-MM-DD]",
  region: "IN"
}
```
This single call identifies, validates and logs all foods automatically.

#### Option B — Search then Log (Standard flow)
```
Step 1: Call FatSecret:search_foods
Input: { query: "[food name]", region: "IN", max_results: 5 }

Step 2: Call FatSecret:get_food  
Input: { food_id: "[id from search results]" }

Step 3: Call FatSecret:log_food
Input: { 
  food_id: "[id]", 
  serving_id: "[default serving_id from get_food]",
  quantity: [number],
  meal: "[meal type]",
  date: "[YYYY-MM-DD]"
}
```

**After logging, ALWAYS call:**
```
Call: FatSecret:get_day_summary
Input: { date: "[YYYY-MM-DD]", calorie_target: 1650 }
```
This gives updated macro totals for the table.

---

### Step 2: Create the Calorie Table

**Table Header** (bold, above columns):
**Rajiv DD-MM Calorie Counter** or **Jasleen DD-MM Calorie Counter**

**Columns:**
| Food Name | FatSecret Link | Serving | Calories | Protein (g) | Fat (g) | Carbs (g) |
|-----------|----------------|---------|----------|-------------|---------|-----------|

**Rules:**
- Food Name must be EXACTLY as returned by FatSecret API (not paraphrased)
- FatSecret Link must be the `food_url` from API response (clickable verification)
- Mark data source: ✅ FatSecret API (Global Database)
- Never use Claude's internal nutritional estimates

---

### Step 3: Add Data Source Citation

Below each food item, always include:
```
🔗 Source: [FatSecret URL] | ✅ Live FatSecret API — verified data
```

Example:
```
Chicken Biryani (Generic) — 208 kcal per 100g
🔗 https://foods.fatsecret.com/calories-nutrition/generic/chicken-biryani
✅ Retrieved live from FatSecret REST API (Global Database, includes Indian foods)
```

---

### Step 4: Date Management
- Same table for the whole day (unless user specifies a historical date)
- New table starts each new day
- Historical dates: pull existing diary via `FatSecret:get_diary` then add to it

---

### Step 5: Total Row (Second Last Row)
Show totals across all columns:
| **TOTAL** | — | — | **sum cal** | **sum P** | **sum F** | **sum C** |

---

### Step 6: Percentage Row (Last Row)
Show:
- **Calories:** X / 1650 (or 2100 weekend) = **XX%** of daily target
- **Remaining:** XXX kcal left
- **Macro split:** Protein XX% | Fat XX% | Carbs XX%

---

### Step 7: Weekly Comparison
After 7 days of tracking, show a 7-day comparison table showing daily totals vs targets.

---

### Step 8: Warning System
- Within 10% of target → No warning
- Exceeded by >10% → **⚠️ WARNING:** "You have exceeded today's target by XXX kcal. Tomorrow's adjusted target: XXXX kcal."

---

### Step 9: Delete Flow
When user says `delete R [entry_id]` or `delete J [entry_id]`:
```
1. Call FatSecret:delete_food_entry with { entry_id: "[id]" }
2. Call FatSecret:get_day_summary to get updated totals
3. Show updated table with the entry removed
```

### Step 10: Update Flow
When user says `update R [entry_id] [qty] [meal]`:
```
1. Call FatSecret:get_diary to get current serving_id for that entry
2. Call FatSecret:update_food_entry with { entry_id, serving_id, quantity, meal }
3. Call FatSecret:get_day_summary to get updated totals
4. Show updated table
```

### Step 11: Day Summary View
When user says `day R`, `summary R`, `day J`, `summary J`:
```
Call: FatSecret:get_day_summary
Input: { date: "[today YYYY-MM-DD]", calorie_target: 1650 }
```
Display the structured response as the calorie table.

### Step 12: Weight Logging
When user says `weight R 72.5`:
```
Call: FatSecret:log_weight
Input: { weight_kg: 72.5, date: "[today]" }
```
Confirm: "⚖️ Weight 72.5 kg logged for Rajiv on DD-MM"

---

### Step 13: Response Format
- On success: **"Calorie Table updated for [Rajiv/Jasleen] DD-MM"** then show the table
- Always include FatSecret URLs for every food item
- Always show the data source attribution line

---

## Example Prompts and Expected Actions

| User Types | Skill Does |
|-----------|-----------|
| `add R 2 rotis and dal lunch` | search_foods("roti") + search_foods("dal tadka") → log_food × 2 → get_day_summary → show table |
| `add J poha 200g breakfast` | search_foods("poha") + get_food → log_food → get_day_summary → show table |
| `calorie R 13-06` | get_diary(date=2026-06-13) → format as table |
| `day R` | get_day_summary(today) → show full summary |
| `delete R 12345678` | delete_food_entry("12345678") → get_day_summary → show updated table |
| `weight R 73.2` | log_weight(73.2) → confirm |
| `summary J` | get_day_summary(today, target=1650) → show table |

---

## Result
Every skill invocation produces a Calorie Table with:
- Live FatSecret-verified nutritional data (never estimated by Claude)
- Clickable FatSecret URLs for each food item
- Running totals with % of daily target
- Macro breakdown (Protein / Fat / Carbs %)
- Separate tracking for Rajiv and Jasleen
