---
name: grocery-list
description: Track the user's grocery list. Add, remove, and check off items. Display items grouped by category.
---

# Grocery List

Manage the user's grocery list in `/workspace/group/grocery_list.json`. Always use this exact path.

## Format

```json
[
  { "id": 1, "text": "Apples", "category": "Produce", "checked": false },
  { "id": 2, "text": "Whole milk", "category": "Dairy", "checked": false },
  { "id": 3, "text": "Chicken breast", "category": "Meat", "checked": true },
  { "id": 4, "text": "Olive oil", "category": "Pantry", "checked": false }
]
```

## Categories

Common categories: **Produce**, **Dairy**, **Meat**, **Pantry**, **Frozen**, **Bakery**, **Beverages**, **Household**, **Personal Care**.

When the user specifies a category, use it exactly. When they don't, infer the best fit from the item name. New categories are fine — use whatever makes sense for the item.

Display categories in this preferred order when present: Produce, Dairy, Meat, Pantry — then any remaining categories alphabetically.

## Rules

- Assign each new item a unique integer `id` (max existing id + 1).
- When marking an item checked, set `checked: true`.
- When marking an item unchecked, set `checked: false`.
- Remove items only when the user explicitly asks to remove or delete them.
- When listing the grocery list, group items by category. Within each group, show unchecked items first, then checked items (marked with ✓). Omit empty categories.
- Omit checked items from the list unless the user asks to see them.

## When to use

- User says "add X to my grocery list", "I need to buy X", "put X on the grocery list"
- User says "what's on my grocery list", "show my groceries", "what do I need to buy"
- User says "I bought X", "got X", "check off X", "remove X from the list"
- User says "clear my grocery list", "start a new list"
