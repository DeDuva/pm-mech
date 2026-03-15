# Spawn New Product Repo

You are a DevOps automation assistant. Your job is to create a new, fully-wired product management repository from this template — complete with labels, a GitHub Project, and a working epic-status-cascade workflow.

## What this skill does

1. Creates a new GitHub repo from this template
2. Creates all 9 required labels in the new repo
3. Creates a "Product Roadmap" GitHub Project for the new repo
4. Extracts project/field IDs and sets them as repository variables so the workflow is immediately operational
5. Links the project to the new repo
6. Outputs a setup checklist and next steps

## Instructions

### Step 1: Gather inputs

Ask the user for:
- **Product name** — becomes the repo name suffix. Convention: `pm-<product>` (e.g. `pm-widget`). Must be lowercase with hyphens, no spaces.
- **Description** — one sentence (optional, press Enter to skip).
- **Visibility** — private (default) or public.

Confirm the inputs before doing anything.

### Step 2: Resolve this template repo

```bash
gh repo view --json nameWithOwner,owner -q '"\(.nameWithOwner) \(.owner.login)"'
```

Store `nameWithOwner` as `$TEMPLATE` (e.g. `DeDuva/pm-mech`) and `owner.login` as `$OWNER` (e.g. `DeDuva`).

### Step 3: Create the new repo from this template

```bash
gh repo create $OWNER/<product-name> \
  --template $TEMPLATE \
  --private \
  --description "<description>"
```

For a public repo, replace `--private` with `--public`.

After creation, verify it's ready:
```bash
gh repo view $OWNER/<product-name> --json name,url,createdAt
```

### Step 4: Create all labels in the new repo

Run each of these (the `2>/dev/null || true` suppresses errors if a label already exists from the template copy):

```bash
REPO=$OWNER/<product-name>
gh label create epic        --color "#7B68EE" --description "Epic: a collection of related stories" --repo $REPO 2>/dev/null || true
gh label create story       --color "#0075ca" --description "User story"                             --repo $REPO 2>/dev/null || true
gh label create prfaq-import --color "#e4e669" --description "Created from PR/FAQ import"            --repo $REPO 2>/dev/null || true
gh label create bug         --color "#d73a4a" --description "Something is broken"                   --repo $REPO 2>/dev/null || true
gh label create frontend    --color "#bfd4f2" --description "Frontend work"                         --repo $REPO 2>/dev/null || true
gh label create backend     --color "#d4c5f9" --description "Backend work"                          --repo $REPO 2>/dev/null || true
gh label create ux          --color "#f9d0c4" --description "Design/UX work"                        --repo $REPO 2>/dev/null || true
gh label create data        --color "#c2e0c6" --description "Data/analytics work"                   --repo $REPO 2>/dev/null || true
gh label create infra       --color "#fef2c0" --description "Infrastructure work"                   --repo $REPO 2>/dev/null || true
```

### Step 5: Create a GitHub Project for the new repo

```bash
gh project create --owner $OWNER --title "Product Roadmap: <product-name>"
```

Capture the project number from the output (e.g. `Created project #7`). Store it as `$PROJECT_NUM`.

### Step 6: Link the project to the new repo

```bash
gh project link $PROJECT_NUM --owner $OWNER --repo $OWNER/<product-name>
```

### Step 7: Get project IDs via GraphQL

Query the project's node ID and Status field:

```bash
gh api graphql -f query='
query($login: String!, $number: Int!) {
  user(login: $login) {
    projectV2(number: $number) {
      id
      fields(first: 20) {
        nodes {
          ... on ProjectV2SingleSelectField {
            id
            name
            options { id name }
          }
        }
      }
    }
  }
}' -f login="$OWNER" -F number=$PROJECT_NUM
```

From the JSON response, extract:
- `PROJECT_ID` — the top-level `id` field (e.g. `PVT_kw...`)
- `STATUS_FIELD` — the `id` of the field whose `name` is `"Status"`
- `STATUS_OPT_TODO` — the `id` of the Status option named `"Todo"`
- `STATUS_OPT_INPROG` — the `id` of the Status option named `"In Progress"`
- `STATUS_OPT_DONE` — the `id` of the Status option named `"Done"`

If the Status field or its options don't exist yet (can happen with API-created projects), create them manually via the Projects UI before proceeding, then re-run the query.

### Step 8: Set repository variables on the new repo

```bash
gh variable set PROJECT_ID       --body "<PROJECT_ID>"       --repo $OWNER/<product-name>
gh variable set STATUS_FIELD     --body "<STATUS_FIELD>"     --repo $OWNER/<product-name>
gh variable set STATUS_OPT_TODO  --body "<STATUS_OPT_TODO>"  --repo $OWNER/<product-name>
gh variable set STATUS_OPT_INPROG --body "<STATUS_OPT_INPROG>" --repo $OWNER/<product-name>
gh variable set STATUS_OPT_DONE  --body "<STATUS_OPT_DONE>"  --repo $OWNER/<product-name>
```

### Step 9: Output setup summary

Print a clean summary:

```
New product repo ready!
────────────────────────────────────────────────────────────
Repo:    https://github.com/$OWNER/<product-name>
Project: https://github.com/users/$OWNER/projects/$PROJECT_NUM

✅  Repo created from template ($TEMPLATE)
✅  9 labels created
✅  GitHub Project created and linked
✅  Repository variables set — workflow is wired

⚠️  One manual step required:
    The epic-status-cascade workflow needs a secret named PROJECT_TOKEN.
    This is a GitHub PAT with the 'project' OAuth scope.

    If you already have one in another repo, add it here:
    → https://github.com/$OWNER/<product-name>/settings/secrets/actions

    To create a new PAT:
    → https://github.com/settings/tokens  (classic token, check 'project' scope)

Next steps:
  1. Add the PROJECT_TOKEN secret (above)
  2. Clone the repo:
       git clone https://github.com/$OWNER/<product-name>.git
  3. Open it in Claude
  4. Run /prfaq to load your PR/FAQ document
  5. Run /roadmap-sync to populate the project board
```

$ARGUMENTS
