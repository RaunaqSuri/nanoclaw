---
name: todo
description: Track to-do items for the user. Add, complete, and list tasks. Automatically purge items completed more than 30 days ago.
---

# To-Do List

Manage the user's to-do list in `/workspace/group/todos.json`. Always use this exact path.

## Format

```json
[
  { "id": 1, "text": "Buy groceries", "done": false, "doneAt": null },
  { "id": 2, "text": "Fix leaky faucet", "done": true, "doneAt": "2026-02-01T10:00:00Z" }
]
```

## Rules

- Assign each new item a unique integer `id` (max existing id + 1).
- When marking an item done, set `done: true` and `doneAt` to the current ISO timestamp.
- When listing items, **delete** any item where `done` is true and `doneAt` is more than 30 days ago, then save the file.
- When marking an item as not done, set `done: false` and `doneAt: null`.
- Show pending items first, then recently completed items.

## When to use

- User says "add a todo", "remind me to", "I need to", "put X on my list"
- User says "what's on my list", "show my todos", "what do I need to do"
- User says "done with X", "finished X", "check off X", "completed X"
