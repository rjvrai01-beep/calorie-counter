---
name: calorie-counter
description: >
  Track daily calorie and macro intake for Rajiv and Jasleen using LIVE verified data
  from the FatSecret MCP server at https://rajivfood.duckdns.org/mcp.

  TRIGGER PHRASES:
  ADD:    "add R", "+ R", "add Rajiv", "log R", "add J", "+ J", "add Jasleen", "log J"
          or just "add [food]" with no R/J → ask who then continue
  DATE:   "add R to dd-mm [food]", "add J to dd-mm [food]"
  VIEW:   "calorie R dd-mm", "calorie J dd-mm", "day R", "day J", "summary R", "summary J"
  DELETE: "delete R [entry_id]", "delete J [entry_id]"
  UPDATE: "update R [entry_id]", "update J [entry_id]"
  WEIGHT: "weight R [kg]", "weight J [kg]"
---

# Calorie Counter Skill — FatSecret MCP v5

## Purpose
Track daily food for Rajiv and Jasleen using LIVE FatSecret data via
`https://rajivfood.duckdns.org/mcp`. All nutrition values, confidence scores
and verification links come from this MCP server — never from Claude's own knowledge.

---

## ABSOLUTE RULES — NEVER BREAK

1. **ALWAYS call FatSecret MCP tools first** — never use Claude's own food data
2. **TABLE_ROW values from the server must be copied EXACTLY** — never modify them
3. **Every table row must have a Confidence column** — values from TABLE_ROW only
4. **Every table row must have a FatSecret Link column** — values from TABLE_ROW only
5. **If FatSecret returns no result** — say "not found in FatSecret" — never estimate
6. **NEVER say "the skill doesn't support X"** — if a tool exists for it, call the tool
7. **Delete = call `FatSecret:delete_food_entry`** — never just redraw table without it
8. **This skill requires Claude Sonnet or Opus** — Haiku cannot reliably follow these steps

---

## Daily Calorie Targets
| Person  | Weekday (Mon–Fri) | Weekend (Sat–Sun) |
|---------|-------------------|-------------------|
| Rajiv   | 1,650 kcal        | 2,100 kcal        |
| Jasleen | 1,650 kcal        | 2,100 kcal        |

---

## Step 0 — Identify Person

- If "R", "Rajiv", "J", or "Jasleen" found → proceed to Step 1
- If NOT found → ask **"For Rajiv or Jasleen? (R/J)"**
  - Remember the full original food command
  - When they reply R or J → continue with their original command automatically
  - Do NOT ask them to retype food

---

## Step 1 — Parse Input

Extract from user message:
- **Person:** R/Rajiv or J/Jasleen
- **Meal:** breakfast / lunch / dinner / other (infer from time if not stated)
- **Foods + quantities** in ANY format:
  - `add R 100 gm rice` → rice, 100g
  - `add R rice 100g` → rice, 100g
  - `add R 2 rotis and dal` → roti × 2, dal × 1
  - `add J 1 cup poha` → poha, 1 cup
  - `add R chicken biryani 250gm` → chicken biryani, 250g
  - `add J poha` → poha, default serving

Build:
- `foods` list: individual food names → e.g. `["rice"]` or `["roti", "dal"]`
- `qty_map`: quantity per food → e.g. `{"rice": "100g"}` or `{"roti": "2 pieces"}`

---

## Step 2 — Call FatSecret MCP (MANDATORY — NEVER SKIP)

```
Call: FatSecret:search_multi_food
Input: {
  foods:   ["rice"],           ← individual food names only
  qty_map: {"rice": "100g"},   ← quantities mapped per food
  region:  "IN"
}
```

The server returns structured blocks including:
- `TABLE_ROW:` — pre-formatted markdown table row (copy this DIRECTLY)
- `CONFIDENCE:` — score/100 + emoji
- `FATSECRET_URL:` — verification link
- `STATUS:` — AUTO_LOG, CONFIRM, or MANUAL_PICK

**Act on STATUS:**
- `AUTO_LOG` → copy TABLE_ROW into calorie table, then call log_food
- `CONFIRM`  → show user: "I found '[food name]' for '[query]' — correct? (y/n)"
               If yes → copy TABLE_ROW_IF_CONFIRMED, call log_food
- `MANUAL_PICK` → show ALT options by number, wait for user choice

**Then call log_food for each confirmed item:**
```
Call: FatSecret:get_food { food_id: "[id from search result]" }
→ gets serving_id and exact per-serving nutrition

Call: FatSecret:log_food {
  food_id:    "[id]",
  serving_id: "[default serving_id from get_food]",
  quantity:   [number],
  meal:       "[meal]",
  date:       "[YYYY-MM-DD]"
}
```

**Finally always call:**
```
Call: FatSecret:get_day_summary {
  date:           "[YYYY-MM-DD]",
  calorie_target: 1650
}
```
Use the TOTALS_ROW and PCT_ROW from this response for the table footer.

---

## Step 3 — Build the Calorie Table

**Header (bold):**
**Rajiv DD-MM Calorie Counter** or **Jasleen DD-MM Calorie Counter**

**Exact column order — always 8 columns (# is row number prefix):**

| # | Food Name | Qty | Calories | Protein (g) | Fat (g) | Carbs (g) | Confidence | FatSecret Link |
|---|-----------|-----|----------|-------------|---------|-----------|------------|----------------|

**Confidence column:**
- Copy EXACTLY from `TABLE_ROW` in server response
- Format: `88/100 ✅` or `65/100 ⚠️` or `35/100 ❌`
- HIGH ✅ = exact or near-exact match
- MEDIUM ⚠️ = possible match — user confirmed
- LOW ❌ = weak match — user manually picked

**FatSecret Link column:**
- Copy EXACTLY from `TABLE_ROW` in server response
- Renders as: `[🔗 View](https://foods.fatsecret.com/...)`
- This is the live FatSecret URL for the user to verify independently

**Nutrition values:**
- Copy EXACTLY from `TABLE_ROW` in server response
- These are FatSecret API values — never substitute with Claude's estimates

---

## Step 4 — Total Row

Copy TOTALS_ROW from `get_day_summary` response EXACTLY:
```
| **TOTAL** | — | — | — | — | **ΣCal** | **ΣP g** | **ΣF g** | **ΣC g** |
```

---

## Step 5 — Percentage Row

Copy PCT_ROW from `get_day_summary` response EXACTLY:
```
| **% of 1650** | — | — | — | — | **XX%** | **P:XX%** | **F:XX%** | **C:XX%** |
```

Then on a new line below the table:
**Remaining: XXX kcal**
✅ FatSecret REST API — Global Database (Live)

---

## Step 6 — Warning

If `get_day_summary` response contains `WARNING:` line → copy it below the table as-is.

---

## Step 7 — Date & History

- Same table maintained all day per person — new foods appended to bottom
- New day → fresh table
- Historical `add R to DD-MM food`:
  → Call `get_diary(date)` first → loads existing TABLE_ROWs → append new food

---

## Step 8 — View Commands

```
"day R" / "summary R" / "calorie R today":
→ Call get_day_summary(today, 1650)
→ Build table using ALL_ITEMS_TABLE_ROWS from response
→ Show with TOTALS_ROW and PCT_ROW

"calorie R DD-MM":
→ Call get_diary(date)
→ Build table using TABLE_ROW entries from response
```

---

## Step 9 — Delete

**Delete triggers** — any of these phrases mean the user wants to remove a food:
- "delete R coffee", "remove R coffee", "delete J rice"
- "delete R [entry_id]"
- "remove coffee from my diary"
- "I don't want coffee logged"

**MANDATORY delete flow — always follow these exact steps:**

```
IF user gives a food name (not an entry_id):
  Step A: Call FatSecret:get_diary { date: today }
          → Find the entry where food_entry_name matches the food name
          → Get its food_entry_id

  Step B: Confirm with user:
          "I found [food name] with entry_id [id] — delete it? (y/n)"

  Step C: If yes → Call FatSecret:delete_food_entry { entry_id: "[id]" }

IF user gives an entry_id directly:
  Step A: Call FatSecret:delete_food_entry { entry_id: "[id]" }

ALWAYS after deletion:
  Call FatSecret:get_day_summary { date: today, calorie_target: 1650 }
  Show updated table with the deleted row removed
  Show updated TOTAL and % rows
```

**NEVER** handle deletion by manually redrawing the table without calling `delete_food_entry`.
**NEVER** say "the skill doesn't have a delete function" — it does, via `FatSecret:delete_food_entry`.

---

## Step 10 — Update

```
"update R [entry_id] [new_qty] [meal]":
→ Call get_diary to get serving_id for that entry
→ Call update_food_entry(entry_id, serving_id, new_qty, meal)
→ Call get_day_summary to refresh
→ Show updated table
```

---

## Step 11 — Weight

```
"weight R 72.5":
→ Call log_weight(72.5, today)
→ Reply: "⚖️ 72.5 kg logged for Rajiv on DD-MM"
```

---

## Response Format

Every successful add:
1. **"Calorie Table updated for [Rajiv/Jasleen] DD-MM"**
2. The 9-column table (all values from FatSecret MCP)
3. **Remaining: XXX kcal**
4. `✅ FatSecret REST API — Global Database (Live)`
5. Warning if applicable
6. Nothing else

---

## Example

**User types:** `add R 100 gm rice lunch`

**Skill does:**
1. Person=Rajiv, Meal=lunch, Food=rice 100g
2. Calls `search_multi_food(foods=["rice"], qty_map={"rice":"100g"}, region="IN")`
3. Server returns TABLE_ROW: `| White Rice (Cooked) | 100g | 91/100 ✅ | [🔗 View](https://foods.fatsecret.com/calories-nutrition/generic/white-rice-cooked) | 130 | 2.7 | 0.3 | 28.0 |`
4. STATUS=AUTO_LOG → copies row directly into table, calls log_food
5. Calls get_day_summary → gets TOTALS_ROW and PCT_ROW
6. Shows:

**Rajiv 15-06 Calorie Counter**

| # | Food Name | Qty | Calories | Protein (g) | Fat (g) | Carbs (g) | Confidence | FatSecret Link |
|---|-----------|-----|----------|-------------|---------|-----------|------------|----------------|
| 1 | White Rice (Cooked) | 100g | 91/100 ✅ | [🔗 View](https://foods.fatsecret.com/calories-nutrition/generic/white-rice-cooked) | 130 | 2.7 | 0.3 | 28.0 |
| | **TOTAL** | — | — | — | **130** | **2.7** | **0.3** | **28.0** |
| | **% of 1650** | — | — | — | **7.9%** | **P:8%** | **F:2%** | **C:90%** |

**Remaining: 1,520 kcal**
✅ FatSecret REST API — Global Database (Live)
