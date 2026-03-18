---
name: projects
description: "Manage tasks across projects using GitHub Projects boards. Use for planning tasks for the project, viewing tasks, eisenhower view, updating tasks status, marking tasks as done, prioritizing, planning across projects. Invoke for ANY task/project management operation."
---

# GitHub Projects Task Management

Manage tasks across all projects using GitHub Projects boards. Claude runs `gh project` commands directly, using cached field IDs from `~/.claude/skills/projects/config.json`.

## Config

Two config files live alongside this skill:

- **`local.md`** — personal settings (GitHub username, board name). Read this at the start of any session. Not tracked in git.
- **`config.json`** — cached board metadata (field IDs, option IDs) so lookups happen once, not on every command. Not tracked in git.

Read both config files at the start of any operation. If either file is missing, tell the user and run `init` first.

## Board Resolution

Each project's CLAUDE.md specifies its board and project tag:

```
## Task Board
Board: 2
Project: MyProject
```

1. Read the project's CLAUDE.md for `Board` and `Project` values
2. If not specified → use `default_board` from config, no Project tag
3. Announce the board on every operation: `[Board: Name (#N)] action description`

When adding tasks, use the Project tag from the project's CLAUDE.md — agents don't need to guess which project a task belongs to.

## Operations

### init

One-time setup. Read `SETUP.md` (in the skill directory) and follow its steps to walk the user through local config, board creation, and caching.

### cache-board

Fetch and cache field metadata for a board:

```bash
gh project field-list <NUM> --owner @me --format json
```

Parse the JSON to extract field IDs and option IDs. Write/update the board entry in config.json.

### add

**When working in a repo context** (the default), create a real issue and add it to the board. This enables commit linking (`fixes #N`), repo issue tracker visibility, and discussion threads.

```bash
# 1. Create issue on the repo
gh issue create --repo owner/repo --title "Title" --body "Body" --json url --jq '.url'

# 2. Add it to the board
gh project item-add <NUM> --owner @me --url <ISSUE_URL> --format json

# 3. Set fields (Status, Importance, Urgency, Project)
gh project item-edit --id <ITEM_ID> --field-id <FIELD_ID> --project-id <PROJECT_ID> --single-select-option-id <OPTION_ID>
```

**When there's no repo context** ( or for cross-repo tasks), create a draft item directly:

```bash
gh project item-create <NUM> --owner @me --title "Title" --body "Description" --format json
# then set fields with item-edit
```

Defaults when not specified: Status → Todo, Importance → Not Important, Urgency → Not Urgent.

### view

List tasks with optional filters:

```bash
gh project item-list <NUM> --owner @me --format json -L 200
```

Filter with jq by Status, Importance, Urgency, Project, or any combination. Display as a formatted table.

### eisenhower

Fetch all active (non-Done) items and group into quadrants:

| Quadrant | Criteria | Meaning |
|----------|----------|---------|
| **Do First** | Urgent + Important | Handle immediately |
| **Schedule** | Not Urgent + Important | Plan and schedule |
| **Delegate** | Urgent + Not Important | Consider delegating |
| **Later** | Not Urgent + Not Important | Low priority backlog |

### update status / done

Find item by title search, update the Status field:

```bash
gh project item-list <NUM> --owner @me --format json -L 200 | jq -r '.items[] | select(.title | test("SEARCH";"i")) | .id'
gh project item-edit --id <ITEM_ID> --field-id <STATUS_FIELD_ID> --project-id <PROJECT_ID> --single-select-option-id <OPTION_ID>
```

`done` is shorthand for Status → Done.

### update importance / update urgency / set project

Same pattern: find item by title, update the relevant field using IDs from config.

### archive

```bash
gh project item-archive <NUM> --owner @me --id <ITEM_ID>
```

### link-repo

```bash
gh project link <NUM> --owner <GITHUB_USERNAME> --repo <GITHUB_USERNAME>/repo-name
```

Note: `--owner @me` doesn't work for link — use the GitHub username from `local.md`.

### add-project

Add a new option to the Project field. The GraphQL mutation replaces all options, so include every existing option name alongside the new one to avoid deleting them.

Steps:
1. Fetch current option names from the live API: `gh project field-list <NUM> --owner @me --format json`
2. Run `updateProjectV2Field` with all existing names + the new one (each option needs `name`, `color`, and `description`):

```bash
gh api graphql -f query='mutation { updateProjectV2Field(input: { fieldId: "<FIELD_ID>", singleSelectOptions: [{name: "Existing1", description: "", color: GRAY}, {name: "New Project", description: "", color: BLUE}] }) { projectV2Field { ... on ProjectV2SingleSelectField { options { id name } } } } }'
```

3. Re-cache: all option IDs change on every update, so run cache-board and update config.json with the new IDs. Items retain their tags as long as the option names stay the same.

Note: `link-repo` and `add-project` are separate concerns — link-repo connects a repo to the board, add-project creates a tag for grouping tasks.

### claim

Add a real issue from another repo/board to the master board:

```bash
gh project item-add <NUM> --owner @me --url <ISSUE_URL> --format json
```

### open

```bash
open "https://github.com/users/<GITHUB_USERNAME>/projects/<NUM>"
```

## Session Behavior

- Announce every board operation to the user — this keeps the user in control and able to correct course
- When completing tracked work, announce and mark Done
- When discovering new work, announce and add to the board
- The user may interrupt and override any board update

## New Project Workflow

When starting a new project:

1. Add project name as a Project field option (via add-project)
2. Add `## Task Board` to the project's CLAUDE.md:
   ```
   ## Task Board
   Board: 2
   Project: Project Name
   ```
3. Create initial tasks
4. Suggest creating a GitHub repo — but don't assume. Some projects start without one, or have multiple repos created later.

When a repo is created, link it to the board: `gh project link <NUM> --owner <GITHUB_USERNAME> --repo <GITHUB_USERNAME>/repo-name`
