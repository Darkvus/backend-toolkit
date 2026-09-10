# Installation

`backend-dev-kit` is a [Claude Code](https://claude.com/claude-code) plugin. Install it via a plugin marketplace pointing at this repository.

## 1. Add the marketplace

From inside Claude Code:

```
/plugin marketplace add Darkvus/backend-dev-kit
```

Or, if you've cloned the repo locally:

```
/plugin marketplace add /path/to/backend-dev-kit
```

## 2. Install the plugin

```
/plugin install backend-toolkit@backend-dev-kit
```

## 3. Verify

Restart Claude Code (or start a new session) and check that the plugin's agents, commands, and skills are available:

```
/agents
/help
```

You should see the `BE-ddd-planner`, `BE-fastapi-planner`, `BE-djangorestframework-planner`, `BE-daas-planner`, and `BE-planner-orchestrator` agents, along with the commands and skills listed in [README.md](README.md).

## Updating

```
/plugin marketplace update backend-dev-kit
```

## Uninstalling

```
/plugin uninstall backend-toolkit@backend-dev-kit
```
