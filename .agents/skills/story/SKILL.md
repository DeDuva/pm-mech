# Create Story

You are a senior product manager. Create a well-formed GitHub Story issue using industry best practices for the current repository.

## Instructions

1. Parse the arguments for: a story description, and optionally a parent epic issue number (e.g. `--epic 12`).

2. If insufficient detail is provided, ask:
   - Who is the user? (role/persona)
   - What do they want to do?
   - Why? (the business/user value)
   - What are the acceptance criteria? (how do we know it's done?)
   - Which epic does this belong to? (ask for epic issue number)
   - What is the story point estimate? (1, 2, 3, 5, 8, 13)

3. Generate the story using the template below.

4. Use `gh` CLI to:
   - Create the `story` label if it doesn't exist (color: `#0075ca`)
   - Create the issue
   - If a parent epic issue number is provided, add a comment to the epic linking this story: `Part of epic #<n>` and update the epic's Stories checklist if possible
   - Output the issue URL

## Story Issue Template

**Title format:** `[STORY] <Verb> <object> as <persona>` (e.g. `[STORY] Filter search results by date as a researcher`)

**Body:**

```markdown
## User Story
**As a** [persona/role],
**I want to** [action or capability],
**So that** [benefit / business value].

## Context & Background
<!-- Why is this story needed now? What is the user's current pain point? -->

## Acceptance Criteria
<!-- Use Given/When/Then or checklist format. Be specific and testable. -->
- [ ] Given [context], when [action], then [outcome]
- [ ] Given [context], when [action], then [outcome]
- [ ] Edge case: ...

## Definition of Done
- [ ] Code reviewed and approved
- [ ] Unit tests written and passing
- [ ] Acceptance criteria verified by PM or QA
- [ ] No new accessibility regressions
- [ ] Deployed to staging

## Design / Mockups
<!-- Link to Figma, screenshots, or write N/A -->

## Technical Notes
<!-- Any known constraints, suggested approach, or APIs involved. Optional. -->

## Story Points
<!-- 1 | 2 | 3 | 5 | 8 | 13 -->

## Parent Epic
<!-- Part of #<epic-issue-number> -->
```

**Labels to apply:** `story`, plus domain labels (e.g. `frontend`, `backend`, `data`, `ux`, `infra`)

## Example gh CLI commands

```bash
# Resolve current repo (run this first, use the output in subsequent commands)
REPO=$(gh repo view --json nameWithOwner -q '.nameWithOwner')

# Ensure labels exist
gh label create story --color "#0075ca" --description "A user story" --repo $REPO 2>/dev/null || true

# Create the story issue — capture the URL, then parse the number from it
STORY_URL=$(gh issue create \
  --repo $REPO \
  --title "[STORY] <title>" \
  --body "<body>" \
  --label "story")
STORY_NUM="${STORY_URL##*/}"

# Add a comment linking to the epic
gh issue comment <epic-number> \
  --repo $REPO \
  --body "Story created: #${STORY_NUM} — <story-title>"

# Create the native sub-issue relationship (enables epic-status-cascade workflow)
gh api repos/$REPO/issues/<epic-number>/sub_issues \
  -X POST -F sub_issue_id=$STORY_NUM
```

After creating, output the issue number and URL clearly.
