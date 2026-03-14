# Roadmap Sync

You are a senior product manager. Sync the GitHub Issues backlog into a GitHub Project roadmap board for `DeDuva/pm-mech`.

## What this skill does

1. Finds or creates a GitHub Project named "Product Roadmap" for this repo
2. Adds all open epic and story issues to the project
3. Configures project fields for status, priority, and quarter
4. Outputs a summary of what was synced

## Instructions

### Step 1: Check for existing project

```bash
gh project list --owner DeDuva --format json
```

Look for a project named "Product Roadmap". If it exists, use it. If not, create it:

```bash
gh project create --owner DeDuva --title "Product Roadmap"
```

Record the project number from the output.

### Step 2: Fetch all epic and story issues

```bash
gh issue list --repo DeDuva/pm-mech --label "epic" --state open --json number,title,url,labels,milestone
gh issue list --repo DeDuva/pm-mech --label "story" --state open --json number,title,url,labels,milestone
```

### Step 3: Add issues to project

For each issue found, add it to the project:

```bash
# Get the issue's node ID
ISSUE_ID=$(gh issue view <number> --repo DeDuva/pm-mech --json id -q '.id')

# Add to project
gh project item-add <project-number> --owner DeDuva --url "https://github.com/DeDuva/pm-mech/issues/<number>"
```

### Step 4: Report results

Output a clean summary:
```
Roadmap Sync Complete
─────────────────────
Project: Product Roadmap (#<n>)
URL: https://github.com/orgs/DeDuva/projects/<n>

Added to project:
  Epics:  <count>
  Stories: <count>

Tip: Open the project URL to configure views, group by epic,
and set iteration fields for sprint planning.
```

## Roadmap View Recommendations

After syncing, advise the user to configure these views manually in the GitHub Projects UI:

1. **Backlog view** — Table layout, group by `Label` (epic vs story), sorted by priority
2. **Roadmap view** — Roadmap layout, date field = milestone due date, group by epic
3. **Sprint board** — Board layout, column field = Status (Todo / In Progress / Done), filtered to current milestone

## Notes

- GitHub Projects API via `gh` CLI has some limitations. If `gh project item-add` fails, fall back to using `gh api` with GraphQL mutations.
- The `gh project` commands require the `project` OAuth scope — check with `gh auth status`.
- Custom fields (Priority, Quarter, Story Points) must be created manually in the Projects UI or via GraphQL API.

$ARGUMENTS
