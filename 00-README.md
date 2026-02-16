# LeanCut Meal Planner — Skills Pack

A reusable set of standalone **Markdown skill files** for an agentic workflow that turns:
- what you already have in the fridge/pantry,
- dietary constraints (allergies/avoidances),
- cooking-time limits,
- and a **daily protein target**

into a realistic **7‑day meal plan** plus a **shopping list**.

## What’s inside (4 skills + 1 bonus MCP tool)
1. `skill-01-intake-normalize.md` — normalize user inputs (inventory, restrictions, time, protein goal).
2. `skill-02-inventory-structure.md` — parse and structure a messy inventory into a usable schema.
3. `skill-03-generate-7day-plan.md` — generate a 7‑day plan with per‑meal protein targets.
4. `skill-04-validate-and-shopping.md` — validate constraints, flag risks, and produce a shopping list.
5. `skill-05-mcp-nutrition-lookup.md` (**bonus**) — optional MCP tool spec for nutrition lookup.

## Success criteria (high-level)
- **Constraint-safe:** no forbidden ingredients; allergy-safe language; cross‑contamination warnings where relevant.
- **Inventory-first:** prioritizes using items on hand; shopping list only fills gaps.
- **Executable:** meals match the time limit and common kitchen assumptions.
- **Protein-aware:** each day meets (or is within tolerance of) the target.

## Quick usage
Paste your raw inventory + constraints, then invoke:
- Skill 01 → Skill 02 → Skill 03 → Skill 04
Optionally, enable Skill 05 if you have an MCP nutrition server available.

