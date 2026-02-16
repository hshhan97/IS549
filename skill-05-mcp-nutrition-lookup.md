# Skill (Bonus): MCP Tool — Nutrition Lookup & Macro Estimation

## Purpose
Provide an **optional MCP tool** for looking up approximate macros (especially protein) for ingredients and assembled meals, so the plan can report more grounded estimates.

> This skill is *optional* and the core workflow works without it.

## Assumptions
- You have access to an MCP server that exposes nutrition lookup functions.
- If no MCP server is available, fall back to estimates and clearly label them as estimates.

## Tool interface (example MCP spec)
**Server name:** `nutrition-mcp`  
**Tools:**
1. `nutrition.search_food`
   - Input: `{ "query": "chicken breast cooked", "limit": 5 }`
   - Output: list of candidates `{id, name, serving_size, macros_per_serving}`
2. `nutrition.get_food`
   - Input: `{ "id": "<food_id>" }`
   - Output: macros payload (protein, carbs, fat, calories) per 100g and per serving
3. `nutrition.estimate_meal`
   - Input: `{ "ingredients": [{"name":"chicken breast","grams":150}, ...] }`
   - Output: total macros + per-ingredient breakdown

## When to use
- After Skill 03 generates a plan, call MCP to replace `protein_estimate_g` ranges with:
  - a computed value (and confidence band)
- During Skill 04 validation, to more accurately detect protein shortfalls.

## Step-by-step behavior
1. For each meal ingredient list, **map** ingredient names to MCP search queries.
2. Select best match using:
   - exact name match > category match > closest synonym
3. Convert portions into grams (use conservative defaults; document assumptions).
4. Call `nutrition.estimate_meal`.
5. Update:
   - `protein_estimate_g` to numeric + ± band
   - `daily_protein_estimate_g` from summed meals
6. If tool errors or returns low confidence:
   - keep ranges and add a note: “MCP lookup unavailable; estimates used.”

## Constraints & safety
- Never claim medical benefits or prescribe diets.
- If user has a serious allergy, include an explicit reminder to check labels and restaurant/kitchen cross-contamination.
- Respect privacy: do not log personal data into tool calls beyond food names and quantities.

## Example usage
User meal: “Greek yogurt + berries + oats”
Call `nutrition.estimate_meal` and set breakfast protein to ~30g (depending on yogurt portion).

## Failure modes
- **No match**: widen estimate range and mark low confidence.
- **MCP down**: proceed without; note gracefully.
