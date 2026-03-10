---
name: calorie-counter
description: Track daily calorie and macro intake with automatic food analysis and target monitoring for Rajiv and Jasleen. Use this skill when someone says "add R" or "+ R" or "add Rajiv" (for Rajiv), "add J" or "+ J" or "add Jasleen" (for Jasleen), or retrieves history with "calorie R dd-mm", "calorie J dd-mm", "calorie Rajiv dd-mm", or "calorie Jasleen dd-mm".
---

# Calorie Counter Skill

## Purpose
This skill helps track daily food intake by automatically analyzing calories and macronutrients from food names and weights, maintaining separate daily tables for Rajiv and Jasleen with running totals and target monitoring.

## When to Use This Skill
**For Rajiv (Husband):**
- "add R [food item] [quantity]"
- "+ R [food item] [quantity]"
- "add Rajiv [food item] [quantity]"
- "+ Rajiv [food item] [quantity]"
- "add R to dd-mm [food item] [quantity]"
- "add Rajiv to dd-mm [food item] [quantity]"
- "calorie R dd-mm"
- "calorie Rajiv dd-mm"

**For Jasleen (Wife):**
- "add J [food item] [quantity]"
- "+ J [food item] [quantity]"
- "add Jasleen [food item] [quantity]"
- "+ Jasleen [food item] [quantity]"
- "add J to dd-mm [food item] [quantity]"
- "add Jasleen to dd-mm [food item] [quantity]"
- "calorie J dd-mm"
- "calorie Jasleen dd-mm"

## Instructions

### Step 0: Validate User Identification
1. Check if user specified "R", "Rajiv", "J", or "Jasleen" in their command
2. If NOT specified, respond ONLY with: **"Please specify who are you - Rajiv or Jasleen"**
3. DO NOT proceed to create table until user provides identification
4. Once identified, maintain SEPARATE tables for each person

### Step 1: Create Calorie Table
Create a Calorie Table having columns titled:
- Food Name
- Food Weight (gm)
- Calories
- Protein
- Fats
- Carbohydrates

### Step 2: Add Table Header
At the very top row of the table (before column headers), display in BOLD:
- **Rajiv dd-mm Calorie Counter** (if tracking for Rajiv)
- **Jasleen dd-mm Calorie Counter** (if tracking for Jasleen)

### Step 3: Analyze Food Items
Based on the Food Name and Weight (gm) of food, analyze using data from Fatsecret, MyFitnessPal or other verified food sources websites for Indian and US foods to understand the Calories, Protein, Fats and Carbohydrates for the food item.

### Step 4: Date Management
Keep the Table the same for the complete day and start creating a new Table only at the start of a day unless user specifically asks to "add R to dd-mm", "add Rajiv to dd-mm", "add J to dd-mm", or "add Jasleen to dd-mm" - then pull the Table for that person and the date/month as shown by "dd-mm" and add the Food item at the bottom of the List.

User can refer back to tables by saying "calorie R dd-mm", "calorie Rajiv dd-mm", "calorie J dd-mm", or "calorie Jasleen dd-mm".

### Step 5: Total Row
At the second last row whenever the Table is generated, always show the Total across all columns.

### Step 6: Percentage Row
Add another row at the final bottom row showing:
- Percentage (%) of total calories completed in "Calories" column out of daily target:
  - **For Jasleen**: Weekdays (Monday to Friday): 1650 calories, Saturday or Sunday: 2100 calories
  - **For Rajiv**: Weekdays (Monday to Friday): 1650 calories, Saturday or Sunday: 2100 calories
- Distribution of percentage (%) of calories consumed via Proteins, Fats and Carbohydrates

### Step 7: Weekly Comparison
After 7 days of tracking for each person separately, show a comparison of the previous 7 days history showing how that user has faired in meeting or exceeding the calorie requirements.

### Step 8: Warning System
- If daily calorie intake is within 10% of daily calorie intake: Don't give a WARNING
- If exceeding more than 10% of calorie intake on any given day: Give a WARNING saying that tomorrow, user will have to compensate by eating lesser count of calories by however much they have exceeded on this day
  - Example: If daily calorie intake goal is 1650 calories and user consumes 1900 calories, then if it's a weekday on the next day ask user to consume less calories by compensating the difference of calories as setting the target on the next day.

### Step 9: Response Format
Always give a 5 word response if everything is successful saying "Calorie Table created for [Rajiv/Jasleen] dd-mm" and then below it just give the Table. No extra Content in the response.

## Result
Whenever the Skill is called, a Calorie Table is generated with the Columns and Rows listed in the Steps, maintaining separate tracking for Rajiv and Jasleen.
