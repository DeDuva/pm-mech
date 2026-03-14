# pm-mech — Claude Instructions

This repo manages the product backlog and roadmap for our product using GitHub Issues and GitHub Projects.

## Repository Purpose

- **Backlog**: GitHub Issues, structured as Epics and Stories
- **Roadmap**: GitHub Projects board (view at https://github.com/DeDuva/pm-mech/projects)
- **Workflow**: PR/FAQ documents → Epics → Stories → GitHub Issues

## Issue Conventions

### Epics
- Label: `epic`
- Title prefix: `[EPIC]`
- Contain: problem statement, success metrics, scope, child story checklist
- Child stories listed in the body as a task list with issue references

### Stories
- Label: `story`
- Title prefix: `[STORY]`
- Must follow INVEST principles
- Must have a "User Story" (As a / I want / So that) and Acceptance Criteria
- Must reference parent epic: `Part of #<epic-number>`
- After creation, comment on the parent epic to register the story

### Labels
| Label | Color | Purpose |
|---|---|---|
| `epic` | `#7B68EE` | Epic-level issues |
| `story` | `#0075ca` | User story issues |
| `prfaq-import` | `#e4e669` | Created from a PR/FAQ import |
| `bug` | `#d73a4a` | Something is broken |
| `frontend` | `#bfd4f2` | Frontend work |
| `backend` | `#d4c5f9` | Backend work |
| `ux` | `#f9d0c4` | Design/UX work |
| `data` | `#c2e0c6` | Data/analytics work |
| `infra` | `#fef2c0` | Infrastructure work |

## Available Skills

- `/prfaq` — Load a PR/FAQ document and translate it into epics and stories
- `/epic` — Create a single well-formed epic issue interactively
- `/story` — Create a single well-formed story linked to a parent epic
- `/roadmap-sync` — Sync all issues into the GitHub Projects roadmap board

## GitHub CLI

All issue and project operations use `gh` CLI. Repo is `DeDuva/pm-mech`.

Always check that required labels exist before creating issues. Use:
```bash
gh label create <name> --color "<hex>" --description "<desc>" --repo DeDuva/pm-mech 2>/dev/null || true
```

## Quality Standards

Before creating any issue, verify:
1. **Epics**: Has measurable success metrics, clear scope boundaries, and a definition of done
2. **Stories**: Passes INVEST checklist, has testable acceptance criteria, references parent epic
3. **Titles**: Use the correct prefix (`[EPIC]` or `[STORY]`) and active voice
