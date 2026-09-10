---
name: backend-commits
description: "Git commit message conventions: format, types, emojis, and scoping rules. Use when committing code or reviewing commit history."
---

# Commits

We use **conventional commits** together with **gitmoji** for our commits.

## Installation

### Python

1. Install the **gitlint** library with poetry, npm, pip, etc.
   - Documentation: [gitlint installation](https://jorisroovers.com/gitlint/latest/installation/)
2. Install the gitlint hook: `gitlint install-hook`
3. Create the **.gitlint** configuration file:

```ini
[general]
ignore=body-is-missing,body-min-length

[title-match-regex]
(?P<emoji>:([\w\d]+-[\w\d]+):)?(?P<type>feat|fix|test|docs)\((?P<scope>[\w\d_\-]*)\): (?P<description>[\w\d_]*)
```

> **Note:** The order is important. If we put the .gitlint file first and then install the hook, it may give us an error.

### JavaScript

```bash
npm install --global git-conventional-commits
```

## Recommended Plugins

### PyCharm
- Conventional Commit
- Gitmoji Plus: Commit Button

### Visual Studio Code
- [Conventional Commits](https://marketplace.visualstudio.com/items?itemName=vivaxy.vscode-conventional-commits)

## Convention

```
<type>(<scope>): <description>

[optional body]

[footer(s)]
```

Where:

* **type**: Commit type
* **scope**: Scope in which the changes are made
* **description**: Brief description of the commit
* **body** (optional): More detailed explanation of the commit if necessary
* **footer**: One always mandatory footer: `Refs: <Task>` referencing the task for which the commit is made and other optional ones if required

### Example

```
✨ feat(searcher): add travel options endpoint

Refs: SMP-1234
Co-authored-by: John Doe <john@example.com>
```

## Links of Interest

* **conventional commits:** [https://www.conventionalcommits.org/en/v1.0.0/](https://www.conventionalcommits.org/en/v1.0.0/)
* **gitmoji:** [https://gitmoji.dev/](https://gitmoji.dev/)
* **gitlint:** [https://jorisroovers.com/gitlint/latest/installation/](https://jorisroovers.com/gitlint/latest/installation/)
