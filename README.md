# pm-mech

A product management factory built on GitHub Issues and GitHub Projects. Drop in a PR/FAQ document and get a fully structured backlog — epics, user stories, roadmap board, automated status tracking, and a product README — in minutes.

Skills are defined in `.agents/skills/` as plain Markdown — readable and executable by any AI agent, not just Claude Code.

## How it works

```
PR/FAQ document
      │
      ▼
 /spawn (from pm-mech)
      │
      ├── Creates a new GitHub repo from this template
      ├── Sets up 9 labels
      ├── Creates a GitHub Project
      └── Wires the epic-status-cascade workflow
            │
            ▼
  Clone new repo → open in your AI agent
            │
            ▼
       /prfaq
            │
            ├── Parses the PR/FAQ
            ├── Proposes epics (confirms with you)
            ├── Proposes stories per epic (confirms with you)
            ├── Creates all GitHub Issues
            └── Saves PRFAQ.md → auto-runs /readme
                      │
                      ▼
            /readme (auto-called by /prfaq)
                      │
                      ├── Reads PRFAQ.md + issues + project URL
                      └── Commits product README.md to the repo
                                │
                                ▼
                      /roadmap-sync
                                │
                                └── Adds all issues to the Project board
```

Each product lives in its own repo. pm-mech is the factory — you never work in it directly after spawning.

---

## Quickstart: spawn a new product repo

### Prerequisites

- [GitHub CLI](https://cli.github.com/) installed and authenticated (`gh auth login`)
- [Claude Code](https://claude.ai/code) with this repo open
- A GitHub PAT with `project` OAuth scope (needed once — see step 5)

---

### Step 1 — Open pm-mech in Claude Code

```bash
git clone https://github.com/DeDuva/pm-mech.git
cd pm-mech
claude
```

---

### Step 2 — Run `/spawn`

In the Claude Code session:

```
/spawn
```

Claude will ask you for:

| Field | Example | Notes |
|---|---|---|
| Product name | `pm-widget` | Becomes the repo name. Convention: `pm-<product>`. Lowercase + hyphens. |
| Description | `Inventory management for small retailers` | One sentence. Optional. |
| Visibility | `private` | Or `public`. Defaults to private. |

Confirm when prompted. Claude then does the rest automatically:

1. Creates `DeDuva/pm-widget` from this template
2. Creates all 9 labels in the new repo
3. Copies the "pm-mech: Project Template" project to create "Product Roadmap: pm-widget" — including the pre-configured Backlog, Sprint Board, and Roadmap views
4. Queries the project's GraphQL IDs (Status field + Todo/In Progress/Done options)
5. Sets those as GitHub Actions repository variables so the workflow is immediately wired

---

### Step 3 — Add the `PROJECT_TOKEN` secret (one manual step)

The epic-status-cascade workflow needs a GitHub PAT with the `project` scope to update project item statuses. Secrets can't be copied via CLI, so you add this once per repo.

1. [Create a classic PAT](https://github.com/settings/tokens) with the **`project`** scope checked (if you don't already have one)
2. Go to the new repo's secrets page:
   `https://github.com/DeDuva/pm-widget/settings/secrets/actions`
3. Click **New repository secret**
4. Name: `PROJECT_TOKEN`, Value: your PAT

> **Tip:** If you're spawning many products, store the PAT in a password manager. The same token works for all repos — you just paste it into each one.

---

### Step 4 — Clone the new repo and open it in Claude Code

```bash
git clone https://github.com/DeDuva/pm-widget.git
cd pm-widget
claude
```

From this point on, you work entirely inside the new product repo. pm-mech is only needed to spawn future repos.

---

### Step 5 — Run `/prfaq` with your PR/FAQ document

```
/prfaq
```

Your AI agent will ask you to paste your PR/FAQ. It then:

1. Summarizes the product vision and asks you to confirm
2. Proposes 3–8 epics — you approve, rename, or cut
3. Proposes 3–8 stories per epic — you approve, edit, or cut
4. Creates all GitHub Issues (epics first, then stories with parent links)
5. Saves the PR/FAQ text as `PRFAQ.md` and commits it
6. Automatically runs `/readme` to generate a product-specific README

You stay in the loop at every step. Nothing is created until you confirm.

---

### Step 6 — Run `/roadmap-sync` to populate the project board

```
/roadmap-sync
```

This adds all open epics and stories to the GitHub Project so you can view them on a board, roadmap, or table.

Open the project URL (printed at the end) to configure views, set priorities, and assign iterations.

---

## Day-to-day workflow

Once your repo is set up, use these skills from inside the product repo:

| Skill | What it does |
|---|---|
| `/epic` | Interactively create a new epic with full template |
| `/story` | Create a new story linked to a parent epic |
| `/roadmap-sync` | Re-sync any new issues into the project board |
| `/readme` | Regenerate the product README from current backlog |
| `/prfaq` | Load another PR/FAQ (e.g. for a v2 or a new feature area) |

---

## Automated status cascading

The `epic-status-cascade` GitHub Action runs every 30 minutes (and on issue close/reopen). It automatically derives each epic's status from its child stories:

| Stories state | Epic becomes |
|---|---|
| All stories are **Done** | **Done** |
| Any story is **In Progress** or **Done** | **In Progress** |
| All stories are **Todo** | **Todo** |

This means you never manually update epic status — just work the stories.

---

## Issue conventions

### Epics — `[EPIC]` prefix, `epic` label

```
[EPIC] Build user onboarding flow
```

Body includes: problem statement, proposed solution, target users, success metrics, in/out scope, stories checklist, effort estimate, definition of done.

### Stories — `[STORY]` prefix, `story` label

```
[STORY] Complete profile wizard as a new user
```

Body includes: As a / I want / So that, acceptance criteria (Given/When/Then), definition of done, story points, parent epic reference.

Stories must pass [INVEST](https://en.wikipedia.org/wiki/INVEST_(mnemonic)):
**I**ndependent · **N**egotiable · **V**aluable · **E**stimable · **S**mall · **T**estable

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

---

## Repository structure

```
pm-mech/
├── README.md                               ← you are here
├── CLAUDE.md                               ← instructions for Claude Code
├── .agents/
│   └── skills/                             ← agent-agnostic skill definitions
│       ├── spawn/SKILL.md                  ← factory: creates new product repos
│       ├── prfaq/SKILL.md                  ← PR/FAQ → epics + stories + README
│       ├── epic/SKILL.md                   ← creates a single epic
│       ├── story/SKILL.md                  ← creates a single story
│       ├── roadmap-sync/SKILL.md           ← syncs issues into GitHub Projects
│       └── readme/SKILL.md                 ← generates product README
├── .claude/
│   └── commands/                           ← Claude Code slash command proxies
│       ├── spawn.md                        → .agents/skills/spawn/SKILL.md
│       ├── prfaq.md                        → .agents/skills/prfaq/SKILL.md
│       ├── epic.md                         → .agents/skills/epic/SKILL.md
│       ├── story.md                        → .agents/skills/story/SKILL.md
│       ├── roadmap-sync.md                 → .agents/skills/roadmap-sync/SKILL.md
│       └── readme.md                       → .agents/skills/readme/SKILL.md
└── .github/
    └── workflows/
        └── epic-status-cascade.yml         ← auto-derives epic status from stories
```

Skills in `.agents/skills/` are plain Markdown — any AI agent that can read files can execute them. The `.claude/commands/` layer is Claude Code-specific and simply delegates to the same source of truth.

---

## Spawning multiple products

Each product gets its own fully independent repo:

```
pm-mech          ← factory (this repo)
pm-widget        ← spawned from pm-mech
pm-dashboard     ← spawned from pm-mech
pm-mobile-app    ← spawned from pm-mech
```

Spawned repos inherit all skills, labels, and workflow. Their GitHub Projects and Actions variables are independent — changes in one repo don't affect any other.

To spawn another product: open pm-mech in Claude Code and run `/spawn` again.
