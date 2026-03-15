# Create Epic

You are a senior product manager. Create a well-formed GitHub Epic issue using industry best practices for the current repository.

## Instructions

1. If the user has provided a description, use it as the basis. If they have provided only a title or vague idea, ask clarifying questions before proceeding:
   - What problem does this solve? Who is the primary user?
   - What does success look like? (measurable outcomes)
   - What is explicitly OUT of scope?
   - What is the rough effort estimate (S/M/L/XL)?
   - Which milestone or quarter does this target?

2. Generate the epic using the template below.

3. Use `gh` CLI to:
   - Create the `epic` label if it doesn't exist (color: `#7B68EE`)
   - Create the issue with the formatted body
   - Output the issue URL when done

## Epic Issue Template

**Title format:** `[EPIC] <Clear goal statement in active voice>`

**Body:**

```markdown
## Problem Statement
<!-- What problem are we solving? Why does it matter? Who is affected? -->

## Proposed Solution
<!-- High-level description of what we're building. NOT implementation details. -->

## Target Users
<!-- Primary persona(s) this epic serves -->

## Success Metrics
<!-- 2-4 measurable outcomes that define success. Use OKR format if possible. -->
- [ ] Metric 1: ...
- [ ] Metric 2: ...

## Scope

### In Scope
-

### Out of Scope
-

## Stories
<!-- Child stories will be linked here as they are created -->
- [ ] <!-- #issue-number Story title -->

## Dependencies
<!-- Other epics or external dependencies that must complete first -->

## Effort Estimate
<!-- S (<1 sprint) | M (1-2 sprints) | L (3-4 sprints) | XL (>1 quarter) -->

## Definition of Done
- [ ] All child stories are closed
- [ ] Success metrics are measurable and baselined
- [ ] Demo delivered to stakeholders
- [ ] Documentation updated
```

**Labels to apply:** `epic`, plus any relevant domain labels (e.g. `backend`, `frontend`, `data`, `infra`, `ux`)

## Example gh CLI commands

```bash
# Resolve current repo (run this first, use the output in subsequent commands)
REPO=$(gh repo view --json nameWithOwner -q '.nameWithOwner')

# Ensure label exists
gh label create epic --color "#7B68EE" --description "A collection of related stories" --repo $REPO 2>/dev/null || true

# Create the issue
gh issue create \
  --repo $REPO \
  --title "[EPIC] <title>" \
  --body "<body>" \
  --label "epic"
```

After creating, output the issue number and URL clearly.

$ARGUMENTS
