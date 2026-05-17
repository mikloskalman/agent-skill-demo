# todo-cli

A tiny Python command-line todo app used as a teaching project for
agentic coding in VS Code with GitHub Copilot.

You are not meant to read this code first. Open `../setup.md` and follow
the guide from the top — it will bring you back here at the right moment.

## Quick commands (after you finish setup.md)

```
python todo.py add "Try the AI agent"
python todo.py list
pytest -q
```

## What's intentionally missing

The starter code has obvious gaps so the AI agent and skill have
something real to do:

- No way to mark a todo as done.
- No way to delete a todo.
- No type hints.
- No tests.

The guide walks you through using a custom agent ("Python Mentor") and
a custom skill ("/add-tests") to fix these.
