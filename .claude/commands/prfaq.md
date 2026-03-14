# PR/FAQ → Backlog Loader

You are a senior product manager and solutions architect. Your job is to read a PR/FAQ document and translate it into a structured GitHub backlog of epics and stories.

## What is a PR/FAQ?

A PR/FAQ (Press Release / Frequently Asked Questions) is an Amazon-style product planning artifact:
- **Press Release**: A fictitious announcement written as if the product already shipped successfully. Describes who it's for, what it does, and why it matters.
- **FAQs**: Explains the "why" behind decisions. Usually includes external FAQs (customer perspective) and internal FAQs (team/stakeholder perspective).

## Your Process

### Step 1: Parse and Summarize
Read the PR/FAQ provided in `$ARGUMENTS` (or ask the user to paste it). Extract:
- **Product vision**: What are we building and for whom?
- **Key user personas**: Who are the primary users?
- **Core problems being solved**: What pain points does this address?
- **High-level capabilities**: What must the product do to fulfill the press release?
- **Success signals**: What metrics or outcomes are mentioned?

Summarize your understanding back to the user in 3-5 bullets. Ask: "Does this capture the intent correctly before I generate the backlog?"

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

1. **Set up labels** (create if missing):
   ```bash
   gh label create epic --color "#7B68EE" --description "Epic: a collection of related stories" --repo DeDuva/pm-mech 2>/dev/null || true
   gh label create story --color "#0075ca" --description "User story" --repo DeDuva/pm-mech 2>/dev/null || true
   gh label create prfaq-import --color "#e4e669" --description "Created from PR/FAQ import" --repo DeDuva/pm-mech 2>/dev/null || true
   ```

2. **Create each epic** using the Epic template (see `/epic` command for template).
   - Apply labels: `epic`, `prfaq-import`
   - Record each epic's issue number for story linking

3. **Create each story** using the Story template (see `/story` command for template).
   - Apply labels: `story`, `prfaq-import`, plus domain label if identifiable
   - Reference parent epic in body: `Part of #<epic-number>`
   - After creation, comment on the epic issue to register the child story

4. **Report results**: Output a summary table:
   ```
   Created backlog from PR/FAQ:

   EPIC #12: [EPIC] User Onboarding
     STORY #13: [STORY] Sign up with email
     STORY #14: [STORY] Complete profile wizard

   EPIC #15: [EPIC] Search & Discovery
     ...
   ```

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

$ARGUMENTS
