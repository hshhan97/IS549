# Skill: Validate Plan & Produce Final Shopping List

## Purpose
Validate that the generated plan is constraint-safe and produce a clean, consolidated shopping list with priorities and substitutions.

## When to use
After Skill 03 produced `MealPlan7Day`.

## Inputs
- `MealPlanningBrief`
- `StructuredInventory`
- `MealPlan7Day`

## Outputs
`ValidatedMealPlanPackage`:
- `meal_plan` (same structure as input but corrected if needed)
- `validation_report`:
  - `hard_ban_violations` (list)
  - `allergen_risks` (list)
  - `time_violations` (list)
  - `inventory_mismatches` (list) — used but not owned
  - `protein_shortfall_days` (list with delta)
- `shopping_list`:
  - `must_buy` (list with amounts/units if inferable)
  - `nice_to_have` (list)
  - `substitutions` (map)
  - `estimated_cost_note` (optional; qualitative only)

## Step-by-step behavior
1. **Hard ban scan** across all meals (ingredients + titles + steps).
2. **Allergen risk scan**:
   - cross-contamination warnings (e.g., if user has nut allergy and recipe suggests “nut butter alternative”).
3. **Time scan**: flag meals exceeding max cook time.
4. **Inventory fidelity scan**:
   - ensure `ingredients_on_hand` are present in `StructuredInventory.items`.
5. **Protein check**:
   - compute daily totals from `protein_estimate_g`.
   - If shortfall > 10% on a day, add a protein snack or increase portions.
6. **Shopping list consolidation**:
   - combine duplicates, group by category (protein/produce/pantry).
   - mark `must_buy` vs `nice_to_have`.
7. Output corrected plan + shopping list + a short “How to iterate next week” reflection.

## Constraints
- Do not silently change meals; if you modify, note the change in `validation_report`.
- Do not provide medical advice; for severe allergies, advise confirming labels and avoiding cross-contamination.

## Failure modes & handling
- **Violations found**: fix if straightforward (swap protein); otherwise surface as blocking issues.
- **Unverifiable staples**: place in `must_buy` as low-cost staples.
- **User wants brand-specific items**: keep generic and allow user to specify later.

## Example usage
Input plan accidentally includes “shrimp fried rice” with seafood ban.
Output: violation flagged + meal replaced with “chicken fried rice”; shopping list updated.
