# Generate Product README

You are a senior technical writer and product manager. Your job is to generate a polished, product-specific `README.md` for the current repository, based on the PR/FAQ document and current GitHub Issues backlog.

## What this skill does

1. Reads `PRFAQ.md` from the repo root for the product narrative
2. Queries current epics and stories from GitHub Issues
3. Gets the GitHub Project URL
4. Generates a `README.md` tailored to this specific product
5. Commits and pushes the README to the repo

## Instructions

### Step 0: Resolve current repo

```bash
REPO=$(gh repo view --json nameWithOwner -q '.nameWithOwner')
OWNER=$(gh repo view --json owner -q '.owner.login')
REPO_NAME=$(gh repo view --json name -q '.name')
REPO_URL=$(gh repo view --json url -q '.url')
```

### Step 1: Read the PR/FAQ

```bash
cat PRFAQ.md
```

If `PRFAQ.md` exists, parse:
- Product name and tagline (from the Press Release headline)
- Target users (personas)
- Core problems solved
- Key capabilities / FAQ highlights

If `PRFAQ.md` does not exist, ask the user to provide a brief product description or run the `prfaq` skill first.

### Step 2: Fetch the current backlog

```bash
# Get all epics
gh issue list --repo $REPO --label "epic" --state open --json number,title --limit 50

# Get all stories
gh issue list --repo $REPO --label "story" --state open --json number,title,labels --limit 100
```

### Step 3: Get the project URL

```bash
gh project list --owner $OWNER --format json --jq \
  '.projects[] | select(.title | startswith("Product Roadmap")) | {number, title}'
```

The project URL is: `https://github.com/users/$OWNER/projects/<number>`

### Step 4: Generate README.md

Using all gathered information, write a `README.md` to the repo root with this structure:

```markdown
# <Product Name>

> <One-sentence tagline from the press release>

## Overview

<2-3 sentence product description based on the PR/FAQ press release. Who is it for, what does it do, why does it matter.>

**Target users:** <primary personas from the PR/FAQ>

## Quick Start

```bash
# Clone this repo
git clone <repo-url>
cd <repo-name>

# Open in your AI agent (e.g. Claude Code)
claude .
```

Run these skills to manage the backlog:
- `/prfaq` — load a new PR/FAQ and generate epics & stories
- `/epic` — create a single epic issue interactively
- `/story` — create a story linked to a parent epic
- `/roadmap-sync` — sync all issues to the project board
- `/readme` — regenerate this README from the current backlog

## Product Roadmap

📋 **[View the GitHub Project board](<project-url>)**

The board tracks all epics and stories by status (Todo / In Progress / Done).
The `epic-status-cascade` workflow automatically advances epic status as its child stories are completed.

## Current Backlog

### Epics

| # | Epic |
|---|------|
<one row per epic: | #N | [EPIC] Title |>

### Stories

| # | Story | Labels |
|---|-------|--------|
<one row per story: | #N | [STORY] Title | label1, label2 |>

## Sprint Planning

### Starting a sprint
1. Open the [project board](<project-url>)
2. Review the **Backlog** — epics and stories in **Todo**
3. Move the sprint's stories from **Todo** → **In Progress**

### During the sprint
- Update story status on the board as you work
- Close stories when complete: `gh issue close <number> --repo <repo>`
- The `epic-status-cascade` workflow auto-advances the parent epic status

### Ending a sprint
- Review what moved to **Done**
- Move incomplete stories back to **Todo** for the next sprint
- Run `/roadmap-sync` to catch any issues not yet on the board

## Epic Status Cascade

Stories drive epic status automatically via a GitHub Actions workflow:

| Story statuses | Epic status |
|----------------|-------------|
| All Todo | Todo |
| Any In Progress or Done | In Progress |
| All Done | **Done** ✅ |

The workflow runs every 30 minutes and on every issue close/reopen event.

## Source PR/FAQ

<include the full contents of PRFAQ.md verbatim here>

---

*Managed with [pm-mech](https://github.com/DeDuva/pm-mech)*
```

Replace all `<placeholders>` with real values from the data gathered in Steps 0–3. Include the full PRFAQ.md content in the "Source PR/FAQ" section.

### Step 5: Commit and push

```bash
git add README.md
git commit -m "docs: generate product README from PR/FAQ and backlog"
git push
```

### Step 6: Output a confirmation

```
README generated and pushed!
────────────────────────────
File:  README.md
Repo:  <repo-url>

The README includes:
  ✅ Product overview from PR/FAQ
  ✅ <N> epics in backlog table
  ✅ <N> stories in backlog table
  ✅ Project board link
  ✅ Sprint planning guide
  ✅ Full source PR/FAQ
```
