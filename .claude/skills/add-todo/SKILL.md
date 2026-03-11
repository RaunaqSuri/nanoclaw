---
name: add-todo
description: Add to-do list tracking to NanoClaw agents. Tracks tasks, marks them done, and auto-purges completed items after 30 days.
---

# Add To-Do List

Adds a to-do list skill to all container agents. The agent manages a `todos.json` file in each group's workspace using its existing Read/Write tools — no additional dependencies.

## Apply

The skill file lives at `container/skills/todo/SKILL.md`. If it doesn't exist, create it with the content from the todo skill definition.

Rebuild the container and restart:

```bash
./container/build.sh
# macOS: launchctl kickstart -k gui/$(id -u)/com.nanoclaw
# Linux: systemctl --user restart nanoclaw
```

## Verify

Send a message like "add buy groceries to my todo list" and confirm the agent creates `todos.json` and responds with the added item. Then ask "what's on my list" and confirm it reads back the items.
