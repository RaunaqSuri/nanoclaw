---
name: add-grocery-list
description: Add grocery list tracking to NanoClaw agents. Add, remove, and check off items. Displays items grouped by category (Produce, Dairy, Meat, Pantry, and any others).
---

# Add Grocery List

Adds a grocery list skill to all container agents. The agent manages a `grocery_list.json` file in each group's workspace using its existing Read/Write tools — no additional dependencies.

## Apply

The skill file lives at `container/skills/grocery-list/SKILL.md`. If it doesn't exist, create it with the content from the grocery-list skill definition.

Rebuild the container and restart:

```bash
./container/build.sh
# macOS: launchctl kickstart -k gui/$(id -u)/com.nanoclaw
# Linux: systemctl --user restart nanoclaw
```

## Verify

Send a message like "add apples and milk to my grocery list" and confirm the agent creates `grocery_list.json` with the items in the right categories. Then ask "what's on my grocery list" and confirm it responds with items grouped by category.
