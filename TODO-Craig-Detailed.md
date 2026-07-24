# TODO — Detailed Execution Plans

Backing detail for the checklist in `TODO-Craig.md`. Each plan below is
written to be handed to Claude Code (git/CLI work) — cross-reference
`CLAUDE.md` for the tool split. Verified against live GitHub data on
24 July 2026; re-check anything time-sensitive before running.

---

## 1. Export/import the Issues backlog from the archived upstream repo

**Why**: Issues and PR discussion threads are platform data, not git data —
they do **not** transfer with a fork and would be lost if the archived repo
ever became unreachable. Confirmed (24 Jul 2026) the repo is still fully
readable via the GitHub REST API despite being archived — issues, comments,
and PR bodies are all retrievable; the only thing archiving blocks is new
writes (all threads are shown as **locked**, no new comments). No special
access is needed since this is a public repo — Craig doesn't need write
access to `microsoft/coe-starter-kit` to read from it.

**Why not GitHub's own "transfer issue" feature**: `gh issue transfer` /
GitHub's UI transfer only works between repos owned by the same user or org
— not applicable here (Craig doesn't own the Microsoft repo, and it's
archived/read-only regardless). Export-and-recreate is the only path.

### Step 1 — Export (read-only, safe, no auth required for public reads)

Use the GitHub CLI (`gh`) — official tool, already the right one for this,
no need for a bespoke script or third-party dependency for the export step:

```bash
# List all issues (open + closed), basic fields
gh issue list --repo microsoft/coe-starter-kit --state all --limit 5000 \
  --json number,title,body,state,labels,createdAt,closedAt,author,url \
  > archive/upstream-issues/issues-summary.json

# Pull full detail + comments per issue (basic fields above don't include comments)
mkdir -p archive/upstream-issues/detail
for n in $(jq -r '.[].number' archive/upstream-issues/issues-summary.json); do
  gh issue view "$n" --repo microsoft/coe-starter-kit \
    --json number,title,body,state,labels,createdAt,closedAt,author,url,comments \
    > "archive/upstream-issues/detail/$n.json"
  sleep 1   # be polite to the API / avoid rate limits
done
```

Commit `archive/upstream-issues/` to the repo as a permanent historical
snapshot — this is the "doesn't need to be pretty" archival copy the
original TODO called for. A flattened CSV (issue number, title, state,
labels, URL) is a nice-to-have for skimming but the JSON is the source of
truth.

### Step 2 — Curate

Skim the export and mark which issues are still relevant bugs worth carrying
forward (cross-reference the known unresolved bugs already noted in
`KNOWLEDGE-Claude-Craig.md` §4: stale object alerts for deleted flows,
inventory sync failures, BYODL/Dataflow failures in GCC environments,
inactivity-cleanup flow failing to enable — plus the ~34 still-open items
from the March 2026 milestone). Don't recreate everything; recreate what's
still actionable.

### Step 3 — Recreate in the new repo

Since attribution can't be preserved automatically (issues created via the
API/CLI are authored by whichever account's token creates them, not the
original reporter), prefix each recreated issue body with an attribution
block — this is the standard workaround used by cross-account migration
tools:

```bash
gh issue create --repo Craig-Humphrey/coe-starter-kit \
  --title "<original title>" \
  --label "bug" \
  --body "> Originally reported by @<author> in microsoft/coe-starter-kit#<number> (<date>).
> Preserved during CoE Starter Kit archival — full original thread in
> \`archive/upstream-issues/detail/<number>.json\`.

<original body>"
```

Create any missing labels first with `gh label create` so `--label` doesn't
error on labels that don't exist yet in the new repo.

### If the backlog turns out to be large/messy

The scripted `gh` loop above is the right first choice for a modest backlog
(tens of issues) — no new dependency, uses the official CLI. If it proves
too tedious for a much larger set, the open-source
[`NicholasBoll/github-migration`](https://github.com/NicholasBoll/github-migration)
tool handles issues/PRs/comments with history rewriting across accounts.
Don't reach for GitHub Enterprise Importer (GEI) — that's built for
org-to-org enterprise migrations at a scale well beyond this project.

### Timing

Not urgent — the archived repo is currently fully readable — but don't treat
that as permanent. Do this during Phase 1, not deferred indefinitely.

---

## 2. Merge the 3 pending Dependabot branches

**Confirmed (24 Jul 2026)**: all three branches are still live on
`microsoft/coe-starter-kit` and fetchable, despite the repo being archived.
Craig's fork only pulled `main` when it was created, so none of these three
came across — they need to be fetched directly from upstream.

| Branch | PR | Bump |
|---|---|---|
| `dependabot/npm_and_yarn/coe-cli/babel/plugin-transform-modules-systemjs-7.29.4` | #11039 | `@babel/plugin-transform-modules-systemjs` 7.16.7 → 7.29.4 |
| `dependabot/npm_and_yarn/coe-cli/brace-expansion-1.1.14` | #11036 | `brace-expansion` 1.1.11 → 1.1.14 |
| `dependabot/npm_and_yarn/coe-cli/axios-0.31.1` | #11032 | `axios` 0.25.0 → 0.31.1 |

### Step 1 — Add upstream as a remote (Claude Code / local git)

```bash
git remote add upstream https://github.com/microsoft/coe-starter-kit.git
```

Read-only use only — never push here (archived repos reject pushes anyway).

### Step 2 — Fetch just these three branches

Fetching by exact name avoids pulling the archived repo's much larger branch
set (hundreds of stale `copilot/...` branches from Microsoft's internal
workflow):

```bash
git fetch upstream dependabot/npm_and_yarn/coe-cli/brace-expansion-1.1.14:upstream-brace-expansion
git fetch upstream dependabot/npm_and_yarn/coe-cli/axios-0.31.1:upstream-axios
git fetch upstream "dependabot/npm_and_yarn/coe-cli/babel/plugin-transform-modules-systemjs-7.29.4":upstream-babel
```

### Step 3 — Check for drift before merging

These branches are 2+ months stale off Microsoft's `main` at archive time,
not off Craig's fork's `main` — confirm they still apply cleanly:

```bash
git diff main upstream-axios -- coe-cli/package.json coe-cli/package-lock.json
```

### Step 4 — Apply the bump (two options)

**Option 1 — cherry-pick the dependency-bump commit (recommended, simpler)**:

```bash
git cherry-pick 66c0a1cf998baa4035694df79fb2280853148e8a   # axios
git cherry-pick 382a958c8fc6c74d92e3cdb0a174df47b49702e5   # brace-expansion
git cherry-pick <babel-branch-tip-sha>                      # babel — get via: git rev-parse upstream-babel
```

If `package-lock.json` conflicts (likely — lockfiles are sensitive to
whatever else has changed), don't fight the merge: apply the version bump to
`package.json` manually, then regenerate the lockfile with
`npm install` inside `/coe-cli` rather than hand-resolving lockfile conflicts.

**Option 2 — full branch merge** (`git merge upstream-axios`, etc.) — more
faithful to "this is literally the original Dependabot PR," but more likely
to hit conflicts if the fork's `main` has diverged at all. Use Option 1
unless there's a specific reason to preserve the exact branch history.

### Step 5 — Security-freshness check (cheap add-on, not in original scope)

These proposed bumps are ~2 months old. After merging, check whether newer
patch versions exist and consider those as the real floor rather than
replaying stale Dependabot suggestions verbatim:

```bash
cd coe-cli && npm outdated && npm audit
```

### Step 6 — Commit with traceability + verify

```bash
git commit -m "Merge dependabot/npm_and_yarn/coe-cli/axios-0.31.1 (was microsoft/coe-starter-kit#11032, unmerged at archive)"
cd coe-cli && npm ci && npm test   # or npm run build / a basic smoke check if no test suite
```

Push to `main` (or a short-lived branch + self-merge PR, if Craig wants a
review step even solo).

---

## 3. Migrate branding from Microsoft to Community CoE Starter Kit

**Constraints** (from `KNOWLEDGE-Claude-Craig.md` §5 + the 24 Jul 2026
decisions): MIT license permits modification/rebranding freely; the original
MIT copyright notice must be retained; "Microsoft" name/logo can't be used in
a way implying endorsement; the new project isn't Fusion5-branded either —
it needs its own clean identity as **Community CoE Starter Kit**.

### Phase A — surface level (do first, cheap, low-risk)

- **Repo settings**: rename the GitHub repo, update its description/topics,
  replace the social preview image if one exists. GitHub auto-redirects the
  old URL, so this is low-risk.
- **README.md**: replace the top Microsoft transition-notice blockquote with
  a Community CoE Starter Kit banner — what this project is, its
  relationship to the archived Microsoft original, and an explicit "not
  affiliated with or endorsed by Microsoft" disclaimer (this disclaimer is a
  trademark requirement, not optional).
- **NOTICE**: add a short `NOTICE.md` (or a clearly separated section in
  `LICENSE`) that keeps the required original MIT copyright line intact and
  adds "this is an independent community continuation, not a Microsoft
  product."
- **CONTRIBUTING.md / CODE_OF_CONDUCT.md**: update Microsoft Open Source
  boilerplate and links back to `microsoft/coe-starter-kit`; drop references
  to the Microsoft CLA (confirmed in `KNOWLEDGE-Claude-Craig.md` §5 that it
  doesn't carry forward) in favor of a lighter note appropriate to a
  solo-maintained-for-now project.

### Phase B — component-level docs (medium effort, after Phase A settles)

- Sweep every `*/README.md`, `SETUPGUIDE.md`, `USERGUIDE.md`, etc. across
  each `CenterofExcellence*` folder for "Microsoft Power Platform Center of
  Excellence" phrasing and replace with "Community CoE Starter Kit" —
  **in prose only**. Do this as `grep -rl` to find hits, then review each one
  by hand rather than a blind `sed` pass: some mentions are legitimate
  factual references to Microsoft/Power Platform (the product this still
  runs on) and should stay as-is.
- In-app branding: Power Apps titles/descriptions/splash/about screens, Power
  BI report titles/cover pages — these live inside the Dataverse solution
  files and are editable via the same solution-unpack tooling `coe-cli`
  already uses. Cosmetic text only at this stage, no schema changes.
- Review any embedded logos/images that used Microsoft or Power
  Platform-branded imagery beyond generic product icons; swap for
  neutral/community assets if there's trademark risk.

### Phase C — solution/schema level (bigger, higher-risk — explicit decision, not a default)

The Dataverse solution(s) currently ship under Microsoft's original
publisher prefix (confirm the exact prefix by inspecting the unpacked
solution's `publisher.xml` — this determines every table/column/flow
internal name). Renaming it is a **breaking change**: managed-solution
upgrade paths rely on matching solution/publisher IDs, so anyone running the
original CoE Starter Kit couldn't upgrade in place onto a republished
solution — they'd need a fresh install.

This needs an explicit decision (add to `SCOPE-CHARTER.md`'s open-decisions
list, don't resolve it silently as part of a "branding cleanup" pass):

- **Keep the original publisher/prefix** — preserves the upgrade path for
  existing CoE Starter Kit installs, but keeps Microsoft's naming baked into
  the schema indefinitely.
- **Re-publish under a new "Community CoE" publisher** — clean brand
  independence, but breaks in-place upgrades for existing installs.

Recommend deferring this to the Phase 2 scope/governance conversation rather
than bundling it into the initial branding pass — bundling it in is exactly
the kind of scope creep flagged as the biggest risk in
`KNOWLEDGE-Claude-Craig.md` §6.
