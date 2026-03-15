# Roadmap Sync

You are a senior product manager. Sync the GitHub Issues backlog into a GitHub Project roadmap board for the current repository.

## What this skill does

1. Finds or creates a GitHub Project named "Product Roadmap" for this repo
2. Adds all open epic and story issues to the project
3. Configures project fields for status, priority, and quarter
4. Outputs a summary of what was synced

## Instructions

### Step 0: Resolve current repo and owner

```bash
gh repo view --json nameWithOwner,owner -q '"\(.nameWithOwner) \(.owner.login)"'
```

Store the `nameWithOwner` value as `$REPO` (e.g. `DeDuva/pm-widget`) and the `owner.login` as `$OWNER` (e.g. `DeDuva`). Use these in all subsequent commands.

### Step 1: Check for existing project

```bash
gh project list --owner $OWNER --format json
```

Look for a project named "Product Roadmap". If it exists, use it. If not, create it:

```bash
gh project create --owner $OWNER --title "Product Roadmap"
```

Record the project number from the output.

### Step 2: Fetch all epic and story issues

```bash
gh issue list --repo $REPO --label "epic" --state open --json number,title,url,labels,milestone
gh issue list --repo $REPO --label "story" --state open --json number,title,url,labels,milestone
```

### Step 3: Add issues to project

For each issue found, add it to the project:

```bash
# Add to project
gh project item-add <project-number> --owner $OWNER --url "https://github.com/$REPO/issues/<number>"
```

### Step 4: Report results

Output a clean summary:
```
Roadmap Sync Complete
─────────────────────
Project: Product Roadmap (#<n>)
URL: https://github.com/users/$OWNER/projects/<n>

Added to project:
  Epics:  <count>
  Stories: <count>

Open the project to see your items across three views:
  • Backlog     — all items in a table
  • Sprint Board — Todo / In Progress / Done columns
  • Roadmap     — timeline view (set milestone dates to populate)
```

## Project Views

GitHub Projects v2 does not support view creation via API — views must be configured manually in the UI. After syncing, advise the user to set up these three views if they haven't already:

1. **Backlog** — click "New view" → Table layout → rename to "Backlog"
   All epics and stories in one place.

2. **Sprint Board** — click "New view" → Board layout → group by **Status**
   Produces Todo / In Progress / Done columns; the `epic-status-cascade` workflow keeps epic cards in sync automatically.

3. **Roadmap** — click "New view" → Roadmap layout
   Assign milestones with due dates to populate the timeline.

Direct link to configure views: `https://github.com/users/$OWNER/projects/<number>/views/1`

## Notes

- GitHub Projects API via `gh` CLI has some limitations. If `gh project item-add` fails, fall back to using `gh api` with GraphQL mutations.
- The `gh project` commands require the `project` OAuth scope — check with `gh auth status`.
- Custom fields (Priority, Quarter, Story Points) must be created manually in the Projects UI or via GraphQL API.
