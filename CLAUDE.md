# pm-mech — Claude Instructions

This is the **product management factory**. It serves two purposes:
1. **Template** — every new product repo is spawned from this one
2. **Active PM repo** — this repo's own issues track the pm-mech product itself

## Repository Purpose

- **Backlog**: GitHub Issues, structured as Epics and Stories
- **Roadmap**: GitHub Projects board (view at the Projects tab for this repo)
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

## Skill Architecture

Skills live in `.agents/skills/<name>/SKILL.md` — agent-agnostic Markdown instructions that any AI agent can read and execute (not just Claude Code). Claude Code slash commands in `.claude/commands/` are thin proxy files that simply delegate to the corresponding `.agents/skills/` file.

```
.agents/skills/
├── spawn/SKILL.md         ← create a new product repo from this template
├── prfaq/SKILL.md         ← parse PR/FAQ → epics & stories → GitHub issues
├── epic/SKILL.md          ← create a single epic issue
├── story/SKILL.md         ← create a single story issue
├── roadmap-sync/SKILL.md  ← sync issues into the GitHub Project board
└── readme/SKILL.md        ← generate the product README from PR/FAQ + backlog

.claude/commands/          ← Claude Code slash command entry points (thin proxies)
├── spawn.md               → delegates to .agents/skills/spawn/SKILL.md
├── prfaq.md               → delegates to .agents/skills/prfaq/SKILL.md
├── epic.md                → delegates to .agents/skills/epic/SKILL.md
├── story.md               → delegates to .agents/skills/story/SKILL.md
├── roadmap-sync.md        → delegates to .agents/skills/roadmap-sync/SKILL.md
└── readme.md              → delegates to .agents/skills/readme/SKILL.md
```

## Available Skills

- `/spawn` — Create a new fully-wired product repo from this template (labels, project, workflow)
- `/prfaq` — Load a PR/FAQ document, translate it into epics and stories, save PRFAQ.md, then auto-run `/readme`
- `/epic` — Create a single well-formed epic issue interactively
- `/story` — Create a single well-formed story linked to a parent epic
- `/roadmap-sync` — Sync all issues into the GitHub Projects roadmap board
- `/readme` — Generate a product-specific README from PRFAQ.md and the current backlog

## GitHub CLI

All issue and project operations use the `gh` CLI. Always resolve the current repo dynamically:

```bash
REPO=$(gh repo view --json nameWithOwner -q '.nameWithOwner')
OWNER=$(gh repo view --json owner -q '.owner.login')
```

Always check that required labels exist before creating issues. Use:
```bash
gh label create <name> --color "<hex>" --description "<desc>" --repo $REPO 2>/dev/null || true
```

## Spawning a New Product Repo

To create a new product management repo for a PR/FAQ:

1. Run `/spawn` from this repo
2. Enter the product name (e.g. `pm-widget`)
3. The skill creates the repo, labels, GitHub Project, and wires the workflow automatically
4. Add the `PROJECT_TOKEN` secret to the new repo (one manual step)
5. Clone the new repo, open it in your AI agent, and run `/prfaq`
6. `/prfaq` will save the PR/FAQ as `PRFAQ.md` and auto-run `/readme` at the end

Each spawned repo is fully self-contained and includes all these same skills.

## Quality Standards

Before creating any issue, verify:
1. **Epics**: Has measurable success metrics, clear scope boundaries, and a definition of done
2. **Stories**: Passes INVEST checklist, has testable acceptance criteria, references parent epic
3. **Titles**: Use the correct prefix (`[EPIC]` or `[STORY]`) and active voice
