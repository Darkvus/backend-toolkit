# Installation

`backend-dev-kit` is a [Claude Code](https://claude.com/claude-code) plugin. Install it via a plugin marketplace pointing at this repository.

## 1. Add the marketplace

From inside Claude Code:

```
/plugin marketplace add Darkvus/backend-toolkit
```

Or, if you've cloned the repo locally:

```
/plugin marketplace add /path/to/backend-toolkit
```

## 2. Install the plugin

```
/plugin install backend-toolkit@backend-toolkit
```

## 3. Verify

Restart Claude Code (or start a new session) and check that the plugin's agents, commands, and skills are available:

```
/agents
/help
```

You should see the `ddd-planner`, `fastapi-planner`, `djangorestframework-planner`, `daas-planner`, and `planner-orchestrator` agents, along with the commands and skills listed in [README.md](README.md).

## Updating

```
/plugin marketplace update backend-toolkit
```

## Uninstalling

```
/plugin uninstall backend-toolkit@backend-toolkit
```
