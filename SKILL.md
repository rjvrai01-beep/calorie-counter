using the basic template as below (---
name: recipe-converter
description: Convert recipes to different serving sizes with accurate ingredient scaling. Use this skill whenever someone wants to adjust recipe portions, scale ingredients up or down, or asks about serving sizes for recipes. Also trigger when they say "double this recipe", "halve the ingredients", or "adjust servings".
---
# Recipe Converter Skill
## Purpose
This skill helps you convert recipe ingredient quantities when changing the number of servings.
## When to Use This Skill
- User uploads or pastes a recipe and wants to change serving size
- User asks to "double", "triple", "halve", or adjust a recipe
- User needs ingredient quantities scaled proportionally
- User asks "how much do I need for X people?"
## Instructions
### Step 1: Identify Original Recipe Details
1. Read the recipe carefully
2. Note the original serving size
3. List all ingredients with their quantities and units
### Step 2: Calculate Conversion Factor), you have to create Frontmatter and Instructions for Calorie Counter using below context (Use Case: Daily Calorie Counter
Trigger: User says - "add to daily chart" or "add calories" or "note calories" or "add further to calories" or "add" or "+" or "add to dd-mm"
Steps: 
1. Create a Calorie Table having columns titled - "Food Name", "Food Weight (gm)", "Calories", "Protein", "Fats", "Carbohydrates"
2. Based on the Food Name, analyze the Food Name and Weight (gm) of food using data from Fatsecret, MyFitnessPal or other verified food sources websites for Indian and US foods to understand what is the Calories, Protein, Fats and Carbohydrates for the food item.
3. Add the Date at the Top row so that I can refer back to the Table by just saying "calorie dd-mm" for each day. Keep the Table the same for the complete day and start creating a new Table only at the start of a day unless I specifically ask to "add to dd-mm" when you pull the Table for the date and month as shown by "dd-mm" and add the Food item at the bottom of the List
4. At the second last row whenever the Table is generated always show the Total across all columns
5. Add another row whenever the Table is generated always show at the final bottom row showing the percentage (%) of total calories completed in "Calories" column completed out of daily target of consuming 1650 calories on Weekdays (Monday to Friday) but if its a Saturday or Sunday the daily target is 2100 calories, and distribution of percentage (%) of calories consumed via Proteins, Fats and Carbohydrates
6. After 7 days of tracking, show a comparison of the previous 7 days history showing how have I faired in meeting or exceeding the calorie requirements
7. If my daily calorie intake is within 10% of my daily calorie intake then dont give me a WARNING, but if I exceed more than 10% of calorie intake on any given day then just give me a WARNING saying that tomorrow, I will have to compensate by eating lesser count of calories by however much I have exceeded on this day, i.e. for example,if my daily calorie intake goal is 1650 calories and I end up consuming 1900 calories then if its a weekday on the next day ask me to consume less calories by compensating the difference of calories as setting my target on the next day.
8. Always give a 5 word response if everything is successful saying "Calorie Table is created for date "dd-mm" and then below it just give the Table. No extra Content in the response.
Result: Whenever the Skill is called a Calorie Table is generated with the Column and Rows listed in the Steps.)
