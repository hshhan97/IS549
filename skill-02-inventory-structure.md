# Skill: Parse & Structure Inventory

## Purpose
Convert a messy inventory list into a structured, deduplicated inventory with categories and “use soon” hints.

## When to use
After Skill 01 produced `MealPlanningBrief.inventory_raw`.

## Inputs
- `MealPlanningBrief` from Skill 01
- Optional: `use_soon_items` (list) if the user calls out expiring foods

## Outputs
`StructuredInventory`:
- `items`: list of
  - `name` (string, canonical)
  - `quantity_hint` (string, optional; e.g., “2 lbs”, “half bag”)
  - `form` (fresh/frozen/canned/dry)
  - `category` (protein/veg/fruit/carb/fat/condiment/snack)
  - `protein_role` (primary/secondary/none)
  - `constraints_risk` (none / check / banned)
- `missing_staples_suspected` (list) — e.g., salt, pepper, oil (mark as “suspected”, not assumed available)
- `notes` (list): dedupe merges, uncertain parses

## Step-by-step behavior
1. **Tokenize** `inventory_raw` by commas, new lines, bullets.
2. **Normalize** names:
   - lowercasing; strip brand words; map common variants
   - examples: “chicken breasts” → “chicken breast”; “greek yogurt” stays
3. **Detect forms**: frozen/canned/dry/fresh via keywords.
4. **Categorize** each item and mark whether it can serve as:
   - primary protein (chicken, tofu, eggs, greek yogurt, beans)
   - secondary protein (cheese, milk, nuts)
5. **Cross-check bans** from `MealPlanningBrief.constraints_normalized.hard_bans`:
   - if banned category detected, set `constraints_risk="banned"` and preserve the item (do not delete), because the user may still have it but must not plan meals with it.
6. Output `StructuredInventory` with clear notes for uncertain items.

## Constraints
- Do not guess quantities if not given; keep `quantity_hint` empty.
- Never “invent” ingredients.
- Keep ambiguous items (e.g., “sauce”) and mark as `constraints_risk="check"`.

## Failure modes & handling
- **User lists meals instead of ingredients**: extract ingredients if possible; otherwise record as “prepared meal” category.
- **Duplicate variants**: merge, keep a note of original strings.
- **Banned items present**: keep them but ensure downstream skills do not use them.

## Example usage
Input inventory: “salmon, eggs, oats, spinach, soy sauce”
Constraints: “no seafood”
Output includes `salmon` with `constraints_risk="banned"`; eggs/oats/spinach allowed.
