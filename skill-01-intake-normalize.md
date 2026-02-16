# Skill: Intake & Normalize Constraints

## Purpose
Turn messy user inputs into a clean, structured **MealPlanningBrief** that downstream skills can reliably use.

## When to use
Use this skill at the start of the workflow whenever the user provides:
- any form of inventory list (text, bullets, screenshots transcribed, etc.)
- dietary constraints (allergies, avoidances, preferences)
- cooking-time limits
- protein goal and/or weight-loss goal

## Inputs
Required:
- `raw_inventory_text` (string): whatever the user typed/pasted about what they have
- `raw_constraints_text` (string): allergies/avoidances/preferences, e.g. “no seafood”, “lactose intolerant”
Optional:
- `daily_protein_target_g` (number) — if missing, ask once; otherwise default to 100g
- `max_cook_time_minutes` (number) — if missing, default to 30
- `servings` (int) — default 1
- `equipment_limits` (string) — e.g., “no oven”, “microwave only”
- `budget_note` (string)

## Outputs
`MealPlanningBrief` (JSON-like object):
- `inventory_raw`
- `constraints_raw`
- `constraints_normalized`: 
  - `allergies` (list)
  - `avoid` (list)
  - `preferences` (list)
  - `hard_bans` (list) — ingredients that must not appear
  - `soft_limits` (list) — minimize but allow if needed
- `targets`:
  - `daily_protein_g`
  - `calorie_style` (e.g., “lean cut”, “maintenance”)
- `limits`:
  - `max_cook_time_minutes`
  - `equipment_limits`
  - `servings`
- `assumptions` (list)
- `questions_needed` (list) — only if critical info is missing

## Step-by-step behavior
1. **Extract constraints** from `raw_constraints_text`.
2. **Normalize synonyms** into canonical forms:
   - “seafood free / no seafood / seafood allergy” → `hard_bans: ["fish", "shellfish", "seafood"]`
   - “vegetarian” → `hard_bans: ["meat", "poultry", "fish", "shellfish"]`
   - “vegan” → add `hard_bans` for eggs/dairy/honey
   - “lactose intolerant” → `soft_limits: ["milk", "cream"]` (unless user says allergy)
3. **Detect ambiguity**:
   - If “allergy” is mentioned, treat as **hard ban** for that ingredient category.
   - If user says “avoid” or “don’t like”, treat as soft limit.
4. **Set defaults** if missing (do not over-question):
   - protein target: 100g/day
   - time limit: 30 minutes
   - servings: 1
5. Produce `MealPlanningBrief` and list any **critical** missing pieces (max 2 questions):
   - “Do you have any *must-use* items expiring soon?”
   - “Confirm your daily protein goal (g/day).”

## Constraints
- Do **not** invent foods or constraints.
- Ask **at most 2** clarifying questions; otherwise proceed with defaults and document assumptions.
- Be explicit that allergy constraints are treated as strict.

## Failure modes & handling
- **Mixed signals** (“I’m vegetarian but sometimes chicken”): record vegetarian as preference, chicken as allowed, and flag in `assumptions`.
- **Unclear allergy** (“seafood sensitivity”): treat as hard ban and note why.
- **No inventory provided**: proceed with a “minimal grocery list starter plan” and clearly label.

## Example usage
User input:
- Inventory: “eggs, greek yogurt, chicken breast, spinach, rice, frozen berries”
- Constraints: “no seafood, lactose intolerant, 20 min meals”
- Protein: 120

Output: `MealPlanningBrief` with `hard_bans=["fish","shellfish","seafood"]`, `soft_limits=["milk","cream"]`, time=20, protein=120.
