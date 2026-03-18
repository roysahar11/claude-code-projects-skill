# GitHub Projects Skill for Claude Code

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that manages tasks across projects using GitHub Projects boards, with Eisenhower matrix prioritization.

## What it does

- **Create and manage tasks** on GitHub Projects boards via `gh` CLI
- **Eisenhower matrix** — tasks are tagged with Importance and Urgency, viewable as quadrants (Do First / Schedule / Delegate / Later)
- **Multi-project support** — tag tasks by project, filter views per project
- **Real issues** — tasks are created as GitHub Issues (linked to repos), not just draft items
- **Cached field IDs** — board metadata is cached locally so every command doesn't re-fetch

## Operations

| Command | What it does |
|---------|-------------|
| `init` | Create a new board with custom fields |
| `cache-board` | Fetch and cache field/option IDs |
| `add` | Create an issue and add it to the board |
| `view` | List tasks with optional filters |
| `eisenhower` | Show active tasks grouped by Eisenhower quadrants |
| `done` | Mark a task as Done |
| `update status/importance/urgency` | Update task fields |
| `set project` | Tag a task with a project |
| `archive` | Archive a board item |
| `link-repo` | Link a GitHub repo to the board |
| `add-project` | Add a new project option to the board |
| `claim` | Add an external issue to your board |
| `open` | Open the board in browser |

## Installation

See **[SETUP.md](SETUP.md)** for the full guide. **Quick start:** clone the repo, symlink it into `~/.claude/skills/projects`, then tell Claude Code `/projects init`.

### Per-project setup

Add a `## Task Board` section to each project's `CLAUDE.md`:

```markdown
## Task Board
Board: 1
Project: MyProject
```

Then tell Claude to add the project tag to the board:

> Add "MyProject" as a project option on the board

## How it works

The skill instructs Claude Code to run `gh project` commands directly. Board metadata (field IDs, option IDs) is cached in `config.json` so commands don't need to re-fetch on every operation.

- `SKILL.md` — the skill definition (instructions for Claude)
- `config.json` — cached board metadata (gitignored, personal)
- `local.md` — your GitHub username and board name (gitignored, personal)

## License

MIT
