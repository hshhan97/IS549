# Skill: Generate 7‑Day High‑Protein Plan (Inventory‑First)

## Purpose
Create a 7‑day meal plan (breakfast/lunch/dinner + optional snack) that:
- uses available inventory first,
- respects constraints,
- fits time/equipment limits,
- hits a daily protein target.

## When to use
After Skills 01–02, once you have:
- `MealPlanningBrief`
- `StructuredInventory`

## Inputs
- `MealPlanningBrief`
- `StructuredInventory`
Optional:
- `meal_slots_per_day` (default: 3 meals + 1 snack)
- `cuisine_preference` (string)

## Outputs
`MealPlan7Day`:
- `days`: list of 7 days each containing:
  - `meals`: list of meal objects:
    - `name`
    - `ingredients_on_hand` (list)
    - `ingredients_to_buy` (list)
    - `cook_time_minutes`
    - `protein_estimate_g` (range allowed)
    - `steps_brief` (5–8 steps max)
    - `leftovers_note` (optional)
  - `daily_protein_estimate_g`
  - `notes` (e.g., substitutions)
- `shopping_candidates` (list) — consolidated but not finalized (Skill 04 will finalize)

## Step-by-step behavior
1. **Compute protein allocation**:
   - breakfast 25–30%
   - lunch 30–35%
   - dinner 30–35%
   - snack 5–15% (optional)
2. **Select primary proteins** from inventory first; rotate to reduce boredom.
3. **Template-driven meal generation**:
   - Choose from safe templates: bowls, stir-fries, salads, wraps, omelets, yogurt bowls, chili, sheet-pan (if oven allowed).
4. **Constraint guardrails**:
   - Hard bans: must not appear in any meal.
   - Soft limits: minimize; prefer substitutes.
5. **Time/equipment guardrails**:
   - If `max_cook_time_minutes` is 20, avoid “slow cook” recipes.
   - If “no oven”, avoid sheet-pan/baking.
6. **Ingredient realism**:
   - If a meal requires staples (oil/salt/pepper), list them under `ingredients_to_buy` unless confirmed on-hand.
7. Produce a full 7‑day plan, and include a short “Why this plan works” summary in `notes` on Day 1.

## Constraints
- No banned ingredients.
- Keep steps brief and replicable.
- Do not claim precise nutrition without a verified lookup (Skill 05). Use *estimates* and ranges.

## Failure modes & handling
- **Inventory too sparse**: create a “starter plan” that repeats core meals and produces a clear shopping list.
- **Conflicting constraints** (e.g., vegan + egg-only inventory): propose substitutions and flag the conflict.
- **Protein goal too high for constraints**: show a realistic plan and note that supplements (e.g., plant protein) may be needed.

## Example usage
Protein: 120g/day, time: 20 min, inventory: chicken + yogurt + eggs + veggies.
Output: 7 days of chicken bowls, egg scrambles, yogurt snacks, with estimated protein per meal.
