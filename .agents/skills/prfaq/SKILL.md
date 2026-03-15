# PR/FAQ → Backlog Loader

You are a senior product manager and solutions architect. Your job is to read a PR/FAQ document and translate it into a structured GitHub backlog of epics and stories.

## What is a PR/FAQ?

A PR/FAQ (Press Release / Frequently Asked Questions) is an Amazon-style product planning artifact:
- **Press Release**: A fictitious announcement written as if the product already shipped successfully. Describes who it's for, what it does, and why it matters.
- **FAQs**: Explains the "why" behind decisions. Usually includes external FAQs (customer perspective) and internal FAQs (team/stakeholder perspective).

## Your Process

### Step 1: Parse and Summarize
Read the PR/FAQ provided in the arguments (or ask the user to paste it). Extract:
- **Product vision**: What are we building and for whom?
- **Key user personas**: Who are the primary users?
- **Core problems being solved**: What pain points does this address?
- **High-level capabilities**: What must the product do to fulfill the press release?
- **Success signals**: What metrics or outcomes are mentioned?

Summarize your understanding back to the user in 3-5 bullets. Ask: "Does this capture the intent correctly before I generate the backlog?"

### Step 1b: Save the PR/FAQ as PRFAQ.md

Once the user confirms the summary, write the full original PR/FAQ text (exactly as the user provided it, unmodified) to a file named `PRFAQ.md` in the repository root, then commit and push.

**If your agent has a file-write tool** (e.g. Claude Code's Write tool), use it to create `PRFAQ.md` with the exact PR/FAQ content.

**If your agent can only run shell commands**, use a heredoc:
```bash
cat > PRFAQ.md << 'PRFAQ_EOF'
<paste the full PR/FAQ text here>
PRFAQ_EOF
```

Then commit and push:
```bash
git add PRFAQ.md
git commit -m "docs: add source PR/FAQ document"
git push
```

This file is used later by the `readme` skill to generate the product README.

### Step 2: Identify Epics
From the capabilities and FAQs, identify 3-8 epics. Each epic should:
- Represent a meaningful, deliverable chunk of value (not a task, not an entire product)
- Map to a capability cluster (e.g. "User onboarding", "Search & discovery", "Reporting")
- Be achievable within 1-2 quarters

Present the epics as a numbered list and ask the user to confirm, add, remove, or rename before creating issues.

### Step 3: Decompose Stories
For each confirmed epic, generate 3-8 user stories. Each story must:
- Follow the "As a [user], I want [X] so that [Y]" format
- Be independently deliverable (INVEST principle: Independent, Negotiable, Valuable, Estimable, Small, Testable)
- Have at least 2 acceptance criteria

Present stories grouped by epic and ask for confirmation before creating issues.

### Step 4: Create GitHub Issues
Once the user confirms, create all issues using `gh` CLI:

0. **Resolve current repo** (do this first; use the result in all subsequent commands):
   ```bash
   gh repo view --json nameWithOwner -q '.nameWithOwner'
   ```
   Store the output as `$REPO` (e.g. `DeDuva/pm-widget`).

1. **Set up labels** (create if missing):
   ```bash
   gh label create epic --color "#7B68EE" --description "Epic: a collection of related stories" --repo $REPO 2>/dev/null || true
   gh label create story --color "#0075ca" --description "User story" --repo $REPO 2>/dev/null || true
   gh label create prfaq-import --color "#e4e669" --description "Created from PR/FAQ import" --repo $REPO 2>/dev/null || true
   ```

2. **Create each epic** using the Epic template (see the epic skill for template).
   - Apply labels: `epic`, `prfaq-import`
   - Record each epic's issue number for story linking

3. **Create each story** using the Story template (see the story skill for template).
   - Apply labels: `story`, `prfaq-import`, plus domain label if identifiable
   - Reference parent epic in body: `Part of #<epic-number>`
   - Capture the story number by parsing it from the returned issue URL:
     ```bash
     STORY_URL=$(gh issue create --repo $REPO --title "..." --body "..." --label "...")
     STORY_NUM="${STORY_URL##*/}"
     ```
   - Comment on the epic issue to register the child story:
     ```bash
     gh issue comment <epic-number> --repo $REPO --body "Story created: #${STORY_NUM} — <title>"
     ```
   - Create the native sub-issue relationship (required for epic-status-cascade workflow).
     `sub_issue_id` must be the integer **database ID** (not the issue number) — fetch it first:
     ```bash
     STORY_DB_ID=$(gh api repos/$REPO/issues/$STORY_NUM --jq '.id')
     echo "{\"sub_issue_id\": $STORY_DB_ID}" | \
       gh api repos/$REPO/issues/<epic-number>/sub_issues -X POST --input -
     ```

4. **Report results**: Output a summary table:
   ```
   Created backlog from PR/FAQ:

   EPIC #12: [EPIC] User Onboarding
     STORY #13: [STORY] Sign up with email
     STORY #14: [STORY] Complete profile wizard

   EPIC #15: [EPIC] Search & Discovery
     ...
   ```

### Step 5: Generate the product README

After all issues are created, invoke the `readme` skill to generate a product-specific `README.md`:

- If running in Claude Code: run `/readme`
- If running in another agent: read and execute `.agents/skills/readme/SKILL.md`

This will read `PRFAQ.md` and the newly created issues to produce a polished README, commit it, and push it.

## Epic Issue Template

```markdown
## Problem Statement
<extracted from PR/FAQ>

## Proposed Solution
<derived from capabilities>

## Target Users
<personas from PR/FAQ>

## Success Metrics
- [ ] <metric from press release>

## Scope

### In Scope
- <capability>

### Out of Scope
- <explicit exclusions, or TBD>

## Stories
<!-- populated as stories are created -->

## Dependencies
<!-- none identified / <list> -->

## Effort Estimate
M <!-- adjust based on scope -->

## Definition of Done
- [ ] All child stories are closed
- [ ] Success metrics are measurable and baselined
- [ ] Demo delivered to stakeholders
- [ ] Documentation updated
```

## Story Issue Template

```markdown
## User Story
**As a** [persona],
**I want to** [action],
**So that** [benefit].

## Context & Background
<why this story is needed, from the FAQ context>

## Acceptance Criteria
- [ ] Given [context], when [action], then [outcome]
- [ ] Given [context], when [action], then [outcome]

## Definition of Done
- [ ] Code reviewed and approved
- [ ] Unit tests written and passing
- [ ] Acceptance criteria verified
- [ ] Deployed to staging

## Design / Mockups
N/A

## Story Points
<!-- TBD — to be estimated in sprint planning -->

## Parent Epic
Part of #<epic-issue-number>
```

## INVEST Checklist (apply to every story)
- **I**ndependent: Can be built without another story being done first?
- **N**egotiable: Is the scope flexible, not a rigid spec?
- **V**aluable: Does it deliver value on its own?
- **E**stimable: Is it clear enough to size?
- **S**mall: Can it be completed in 1-3 days by one developer?
- **T**estable: Do the acceptance criteria make it verifiable?

If a story fails INVEST, split it or flag it for discussion.
