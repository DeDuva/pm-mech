# pm-mech Test Plan

End-to-end verification that the factory pipeline works correctly. Run this plan against a fresh test repo before shipping changes to pm-mech.

**Estimated time:** 30–40 minutes (plus manual browser verification)
**Prerequisites:** `gh` CLI authenticated, Claude Code open on `pm-mech`

---

## Setup

Create a scratch PAT or reuse your existing `PROJECT_TOKEN` PAT. You will need it in TC-02.

All test cases use a throwaway repo named `pm-test-<date>` (e.g. `pm-test-20260314`).

**Artifacts are left in place** after the test run — use TC-10 to verify them in the browser, then delete manually if desired.

---

## TC-01 — `/spawn`: repo creation

**Objective:** Verify the factory creates a fully wired repo from scratch.

**Steps:**
1. Open `pm-mech` in Claude Code
2. Run `/spawn`
3. Enter product name: `pm-test-<today's date>` (e.g. `pm-test-20260314`)
4. Enter description: `Throwaway repo for pm-mech test plan`
5. Visibility: `private`
6. Confirm when prompted

**Expected results:**

| Check | How to verify |
|---|---|
| Repo exists | `gh repo view DeDuva/pm-test-<date> --json name,url` |
| Repo was created from template (has all skill files) | `gh api repos/DeDuva/pm-test-<date>/contents/.claude/commands --jq '.[].name'` → should list `epic.md`, `prfaq.md`, `roadmap-sync.md`, `spawn.md`, `story.md` |
| Workflow file present | `gh api repos/DeDuva/pm-test-<date>/contents/.github/workflows/epic-status-cascade.yml` → 200 OK |
| All 9 labels exist | `gh label list --repo DeDuva/pm-test-<date> --json name --jq '.[].name'` → `epic`, `story`, `prfaq-import`, `bug`, `frontend`, `backend`, `ux`, `data`, `infra` |
| GitHub Project created | `gh project list --owner DeDuva --format json --jq '.[].title'` → includes `Product Roadmap: pm-test-<date>` |
| Project linked to repo | `gh api repos/DeDuva/pm-test-<date>/projects --jq '.[].name'` → `Product Roadmap: pm-test-<date>` |
| All 5 repo variables set | `gh api repos/DeDuva/pm-test-<date>/actions/variables --jq '.variables[].name'` → `PROJECT_ID`, `STATUS_FIELD`, `STATUS_OPT_TODO`, `STATUS_OPT_INPROG`, `STATUS_OPT_DONE` |
| Variable values are non-empty | `gh api repos/DeDuva/pm-test-<date>/actions/variables --jq '.variables[] | "\(.name)=\(.value)"'` → no empty values |

**Pass criteria:** All 8 checks pass with no errors.

---

## TC-02 — `PROJECT_TOKEN` secret and workflow smoke test

**Objective:** Verify the epic-status-cascade workflow runs without errors once the secret is in place.

**Steps:**
1. Add your PAT as `PROJECT_TOKEN` to the test repo:
   `https://github.com/DeDuva/pm-test-<date>/settings/secrets/actions`
2. Manually trigger the workflow:
   ```bash
   gh workflow run epic-status-cascade.yml --repo DeDuva/pm-test-<date>
   ```
3. Wait ~15 seconds, then check the result:
   ```bash
   gh run list --repo DeDuva/pm-test-<date> --limit 1 --json status,conclusion,displayTitle
   ```

**Expected results:**

| Check | Expected |
|---|---|
| Workflow run status | `completed` |
| Workflow run conclusion | `success` |
| No "undefined" or missing variable errors in logs | `gh run view <run-id> --repo DeDuva/pm-test-<date> --log` → no errors |

**Note:** With no issues in the repo yet, the workflow should log something like `No project items found` or complete with 0 epics processed — that is a passing result.

**Pass criteria:** Workflow run completes with `success` conclusion.

---

## TC-03 — Clone and repo-agnostic skill resolution

**Objective:** Verify all skills correctly identify the new repo (not pm-mech) when run from the cloned test repo.

**Steps:**
1. Clone the test repo **into the same parent folder as pm-mech**, then open it in Claude Code:
   ```bash
   # From the pm-mech parent directory (e.g. C:\Users\dovzi\dev):
   cd ..
   git clone https://github.com/DeDuva/pm-test-<date>.git
   cd pm-test-<date>
   claude
   ```
2. In the Claude Code session, run:
   ```bash
   gh repo view --json nameWithOwner -q '.nameWithOwner'
   ```

**Expected result:** Output is `DeDuva/pm-test-<date>` — **not** `DeDuva/pm-mech`.

**Pass criteria:** Repo resolves to the test repo, not the factory.

---

## TC-04 — `/prfaq`: backlog generation

**Objective:** Verify that running `/prfaq` in the test repo creates issues in the test repo, saves `PRFAQ.md`, and does not touch pm-mech.

**Steps:**
1. From the `pm-test-<date>` Claude Code session, run `/prfaq`
2. When prompted for the PR/FAQ, paste the contents of `test_prfaq.md` from the pm-mech repo:
   ```bash
   # From a terminal in pm-mech:
   cat test_prfaq.md
   ```
   Copy the output and paste it into the Claude session.
3. Confirm the proposed epics (accept the defaults)
4. Confirm the proposed stories (accept the defaults)
5. Allow Claude to create all issues

**Expected results:**

| Check | How to verify |
|---|---|
| Epics created in test repo (not pm-mech) | `gh issue list --repo DeDuva/pm-test-<date> --label epic --json number,title` → 3+ epics |
| Stories created in test repo | `gh issue list --repo DeDuva/pm-test-<date> --label story --json number,title` → stories present |
| `prfaq-import` label applied | `gh issue list --repo DeDuva/pm-test-<date> --label prfaq-import --json number` → same count as above |
| Zero issues created in pm-mech | `gh issue list --repo DeDuva/pm-mech --json number --jq length` → same count as before this test |
| Stories reference parent epics | Open any story issue in browser → body contains `Part of #<number>` |
| Epics have story comments | `gh issue view <epic-number> --repo DeDuva/pm-test-<date> --comments` → at least one comment linking child stories |
| `PRFAQ.md` committed to repo | `gh api repos/DeDuva/pm-test-<date>/contents/PRFAQ.md` → 200 OK (file exists) |
| `PRFAQ.md` matches source | `gh api repos/DeDuva/pm-test-<date>/contents/PRFAQ.md --jq '.content' \| base64 -d \| diff - ../pm-mech/test_prfaq.md` → no diff |

**Pass criteria:** Issues land in the test repo only; all structural links are correct; PRFAQ.md is committed.

---

## TC-05 — `/epic`: single epic creation

**Objective:** Verify the `/epic` skill creates a well-formed epic in the current repo.

**Steps:**
1. From the `pm-test-<date>` Claude Code session, run:
   ```
   /epic Add offline mode so users can manage tasks without internet
   ```
2. Answer any clarifying questions Claude asks

**Expected results:**

| Check | How to verify |
|---|---|
| Issue created in test repo | `gh issue list --repo DeDuva/pm-test-<date> --label epic --json number,title` → new `[EPIC]` entry |
| Title starts with `[EPIC]` | Check title in output |
| Body contains all required sections | `gh issue view <number> --repo DeDuva/pm-test-<date>` → Problem Statement, Success Metrics, Scope, Stories, Definition of Done all present |
| `epic` label applied | `gh issue view <number> --repo DeDuva/pm-test-<date> --json labels --jq '.labels[].name'` → includes `epic` |

**Pass criteria:** Issue created with correct structure and labels.

---

## TC-06 — `/story`: single story creation with epic link

**Objective:** Verify the `/story` skill creates a story and registers it on the parent epic.

**Steps:**
1. Note the issue number of any epic from TC-04 or TC-05 (e.g. `#1`)
2. Run:
   ```
   /story --epic 1 Show a sync status badge in the toolbar so users know when they are offline
   ```

**Expected results:**

| Check | How to verify |
|---|---|
| Issue created in test repo | `gh issue list --repo DeDuva/pm-test-<date> --label story` → new `[STORY]` entry |
| Title starts with `[STORY]` | Check title in output |
| Body contains `Part of #1` | `gh issue view <number> --repo DeDuva/pm-test-<date>` |
| Body has Acceptance Criteria | At least 2 Given/When/Then items present |
| Comment added to parent epic | `gh issue view 1 --repo DeDuva/pm-test-<date> --comments` → comment referencing the new story number |

**Pass criteria:** Issue created, parent epic updated.

---

## TC-07 — `/roadmap-sync`: project population

**Objective:** Verify all issues are added to the GitHub Project board.

**Steps:**
1. Run `/roadmap-sync` from the `pm-test-<date>` Claude Code session
2. Note the project URL in the output

**Expected results:**

| Check | How to verify |
|---|---|
| Command completes without errors | No error output from `gh` |
| Project URL matches the one created in TC-01 | Compare URLs |
| All epics appear in the project | Open project URL in browser → filter by `epic` label |
| All stories appear in the project | Filter by `story` label |

**Pass criteria:** Project board is populated with all open issues.

---

## TC-08 — `epic-status-cascade`: status propagation

**Objective:** Verify closing all stories under an epic flips the epic's project status to Done.

**Steps:**
1. Pick one epic from TC-04 and note its issue number (e.g. `#2`)
2. Find all story issues that reference `Part of #2`:
   ```bash
   gh issue list --repo DeDuva/pm-test-<date> --label story --search "Part of #2" --json number,title
   ```
3. If there are no stories on that epic, create one with `/story --epic 2 <any title>`
4. Add the story to the project:
   ```bash
   gh project item-add <project-number> --owner DeDuva --url https://github.com/DeDuva/pm-test-<date>/issues/<story-number>
   ```
5. Close all story issues under that epic:
   ```bash
   gh issue close <story-number> --repo DeDuva/pm-test-<date>
   ```
6. Manually trigger the workflow:
   ```bash
   gh workflow run epic-status-cascade.yml --repo DeDuva/pm-test-<date>
   ```
7. Wait ~20 seconds, then check the epic's project status:
   ```bash
   gh run view --repo DeDuva/pm-test-<date> $(gh run list --repo DeDuva/pm-test-<date> --limit 1 --json databaseId --jq '.[0].databaseId') --log
   ```

**Expected results:**

| Check | Expected |
|---|---|
| Workflow run concludes `success` | `gh run list --repo DeDuva/pm-test-<date> --limit 1 --json conclusion` → `success` |
| Log shows epic status change | Log contains `Epic #2: 'Todo' → 'Done' ✓` (or similar transition) |
| Epic's project card shows "Done" | Open project board in browser → epic card is in the Done column |

**Pass criteria:** Workflow fires, reads the parameterised variables correctly, and cascades status.

---

## TC-09 — `/spawn` idempotency: duplicate name handling

**Objective:** Verify that running `/spawn` with a name that already exists fails gracefully rather than corrupting the existing repo.

**Steps:**
1. From `pm-mech`, run `/spawn` again with the same name: `pm-test-<date>`
2. Observe the error from `gh repo create`

**Expected result:** Claude reports that the repo already exists and stops — it does **not** proceed to create labels, project, or variables again.

**Pass criteria:** Clear error message, no side-effects on the existing test repo.

---

## TC-10 — Manual browser verification

**Objective:** Visually confirm the full end-to-end output looks correct in the GitHub UI.

**This is a manual step — open each URL in your browser.**

### 10a — Repository contents

Open `https://github.com/DeDuva/pm-test-<date>`

| Check | What to look for |
|---|---|
| `PRFAQ.md` present | File listed in the repo root |
| `PRFAQ.md` content | Click the file — should match the contents of `test_prfaq.md` in pm-mech |
| `README.md` updated | If `/prfaq` auto-ran `/readme`, the README should reflect the product, not the pm-mech template |
| `.agents/skills/` present | Directory visible in the file tree |
| `.github/workflows/` present | `epic-status-cascade.yml` listed |

### 10b — Issues

Open `https://github.com/DeDuva/pm-test-<date>/issues`

| Check | What to look for |
|---|---|
| Epics visible | Issues with `[EPIC]` prefix and purple `epic` label |
| Stories visible | Issues with `[STORY]` prefix and blue `story` label |
| `prfaq-import` label | Yellow label on all issues from TC-04 |
| Parent links | Open a story → body contains `Part of #<epic-number>` |
| Story comments on epics | Open an epic → comments section lists child story numbers |

### 10c — GitHub Project board

Open the project URL printed by `/roadmap-sync` (format: `https://github.com/users/DeDuva/projects/<n>`)

| Check | What to look for |
|---|---|
| All epics appear | Cards for every `[EPIC]` issue |
| All stories appear | Cards for every `[STORY]` issue |
| Status column visible | Default "Status" field shown |
| Epic from TC-08 is "Done" | The epic you closed all stories for shows "Done" status |

### 10d — Project views (manual setup, if not already configured)

GitHub Projects v2 does not support view creation via API — set these up once:

1. Click **+ New view** → **Table** layout → rename to **Backlog**
2. Click **+ New view** → **Board** layout → group by **Status** → rename to **Sprint Board**
3. Click **+ New view** → **Roadmap** layout → rename to **Roadmap**

| Check | What to look for |
|---|---|
| Backlog view | All issues visible in a flat table |
| Sprint Board | Todo / In Progress / Done columns with issue cards |
| Roadmap view | Timeline visible (assign milestones with due dates to populate bars) |

**Pass criteria:** All items visible in the UI; project board shows correct statuses; PRFAQ.md matches the source text.

---

## Pass/fail summary

| TC | Area | Result |
|---|---|---|
| TC-01 | `/spawn` repo creation | |
| TC-02 | Workflow smoke test | |
| TC-03 | Repo-agnostic skill resolution | |
| TC-04 | `/prfaq` backlog generation + PRFAQ.md saved | |
| TC-05 | `/epic` single epic | |
| TC-06 | `/story` with parent link | |
| TC-07 | `/roadmap-sync` project population | |
| TC-08 | Epic status cascade | |
| TC-09 | Spawn idempotency | |
| TC-10 | Manual browser verification | |

All 10 TCs must pass before merging changes to `main`.

---

## Known limitations

- **`PROJECT_TOKEN` secret** is not verifiable via CLI (write-only). TC-02 implicitly validates it by running the workflow.
- **Status cascade timing** — the workflow runs on a 30-minute schedule and on `issues.closed`. In TC-08 we trigger it manually to avoid waiting.
- **GitHub Projects UI fields** (Priority, Quarter, Sprint) are not covered here — they require manual setup and have no CLI assertions.
- **Project views** cannot be created via API (GitHub Projects v2 limitation). TC-10d covers manual setup.
- **Artifacts are intentionally left in place** after a test run. Delete manually via `gh repo delete DeDuva/pm-test-<date> --yes` and `gh project delete <n> --owner DeDuva` when you are done reviewing.
