# Setup Guide

One-time setup for the GitHub Projects skill. Follow these steps to create your board and configure the skill.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- [GitHub CLI (`gh`)](https://cli.github.com/) installed and authenticated
- [`jq`](https://jqlang.github.io/jq/) installed (used for filtering board data)
- A GitHub account

## Step 1 — Install the skill

```bash
# Clone the repo
git clone https://github.com/roysahar11/claude-code-projects-skill.git ~/dev/claude-code-projects-skill

# Symlink into Claude Code skills folder
ln -s ~/dev/claude-code-projects-skill ~/.claude/skills/projects
```

If you already have a `~/.claude/skills/projects` folder, back it up or remove it first.

## Step 2 — Create local config

Copy the example and fill in your details:

```bash
cd ~/.claude/skills/projects
cp local.example.md local.md
```

Edit `local.md`:

```markdown
# Local Configuration

GitHub Username: your-github-username
Board Name: My Task Board
```

## Step 3 — Create the board

Run these commands (replace `<BOARD_NAME>` with the name from your `local.md`):

```bash
gh project create --owner @me --title "<BOARD_NAME>" --format json
```

This returns a project number (`<NUM>`). Use it in the next commands:

```bash
gh project field-create <NUM> --owner @me --name "Importance" --data-type "SINGLE_SELECT" --single-select-options "Important,Not Important"
gh project field-create <NUM> --owner @me --name "Urgency" --data-type "SINGLE_SELECT" --single-select-options "Urgent,Not Urgent"
gh project field-create <NUM> --owner @me --name "Project" --data-type "SINGLE_SELECT" --single-select-options "General"
```

## Step 4 — Cache board metadata

Fetch the field IDs and save them to `config.json`:

```bash
gh project field-list <NUM> --owner @me --format json
```

Parse the output and write `config.json` following the structure in `config.example.json`. Or just tell Claude Code:

> Cache board `<NUM>`

and it will do it for you.

## Step 5 — Set the default board

Edit `config.json` and set `"default_board"` to your board number.

## Per-project setup

For each project that should use the board, add a `## Task Board` section to the project's `CLAUDE.md`:

```markdown
## Task Board
Board: 1
Project: MyProject
```

Then tell Claude to add the project tag:

> Add "MyProject" as a project option on the board

## Quick start (let Claude do it)

Alternatively, just tell Claude Code:

> `/projects init`

and it will walk you through all of the above interactively.
