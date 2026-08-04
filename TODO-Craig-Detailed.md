# TODO — Detailed Execution Plans

Backing detail for the checklist in `TODO-Craig.md`. Each plan below is
written to be handed to Claude Code (git/CLI work) — cross-reference
`CLAUDE.md` for the tool split. Verified against live GitHub data on
24 July 2026; re-check anything time-sensitive before running.

---

## 0. Establish a working build/test baseline (do this first)

**Why**: before merging any dependency bump — Dependabot's or a newer
security floor — we need to be able to prove a change still builds and tests
clean. Right now CI can't fully do that. Verified 24 Jul 2026, findings in
`KNOWLEDGE-Claude-Craig.md` §7.

**Current state**:
- `npm install`, `npm run build` (`tsc`), and `npm test` (jest) all work
  today on Node v24.15.0 — 89/89 tests pass, build is clean.
- `.github/workflows/pr-loop-jest-tests.yml` pins **Node 16.x** and only runs
  `npm install` + `npm test` — it does not run `npm run build` or
  `npm run lint`. A change that breaks compilation would still pass CI.
- `npm run lint` is broken independent of any of this: no `.eslintrc*` has
  ever existed for `coe-cli` (checked git history), and the script
  (`eslint . & echo 'lint complete'`) uses `&` not `&&`, so it always
  reports success regardless. Long-standing, not caused by recent drift.
- No `engines` field in `coe-cli/package.json` — nothing pins a supported
  Node range at the package level either.

**Done (24 Jul 2026) — items 1 and 2:**
1. `.github/workflows/pr-loop-jest-tests.yml` now runs `npm run build` and
   `npm run lint` as their own named steps between install and test, so
   either one failing now fails CI (previously only `npm test` ran).
2. `coe-cli/.eslintrc.json` created (`@typescript-eslint/recommended`,
   `no-var`/`prefer-const`/`no-var-requires`/`no-empty-function` turned off
   since they were 100% pre-existing legacy-JS style debt unrelated to
   correctness — see `KNOWLEDGE-Claude-Craig.md` §7 for the before/after
   error counts). Fixed the `lint` script's `&`→command bug that was masking
   failures. Ran `eslint --fix` for the remaining trivial, auto-fixable
   production-code issues (redundant type annotations, `Boolean`→`boolean`);
   the one non-auto-fixable case (`no-this-alias` in `login.ts`) got a scoped
   `eslint-disable-next-line` rather than a behavioral refactor. Result:
   `npm run lint` now exits 0 with 0 errors / 294 warnings (the warnings are
   tracked debt, not blocking — mostly `no-explicit-any`/`no-unused-vars`).
   Also added `eslint` itself as an explicit devDependency (`^8.8.0`) since
   it was previously only resolving transitively.

**Item 3 — done (3 Aug 2026)**: bumped CI's `pr-loop-jest-tests.yml` from
Node `16.x` (EOL) to `24.x` (current LTS) — matches what's already installed
locally, so this closes the version gap outright rather than leaving it as a
decision to make later; also bumped `actions/setup-node` `@v1` → `@v4` in the
same edit. Added `"engines": { "node": ">=20" }` to `coe-cli/package.json`.
Re-verified the full pipeline (`npm run build` / `npm run lint` / `npm test`)
still passes clean post-change: 0 lint errors, 89/89 tests. Considered and
rejected testing against multiple Node versions instead of picking one —
that pattern earns its cost for a published library other people's projects
depend on across environments you don't control; `coe-cli` is an
end-user-installed CLI, not a dependency, so there's no such matrix of
consumers to protect. Single current-LTS version is enough.

**Item 4 — still open (unchanged, not yet actioned)**: optionally add the
build+test+lint steps as a documented contributor workflow section in
`HOW_TO_CONTRIBUTE.md`, since nothing currently tells
   a contributor how to build/test `coe-cli` from source.

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

**Sequencing (decided 24 Jul 2026): do §0 (build/CI baseline) first.** Once
that lands, follow this plan as originally written — bring in the literal
Dependabot bumps below, then run the `npm audit`/OSV re-evaluation as a
separate high-priority follow-up (`TODO-Craig.md`) rather than trying to
leapfrog to fully-patched versions in this same pass. Reasoning and the
specific vulnerability gaps found (`brace-expansion` 1.1.14 and `axios`
0.31.1 both leave known HIGH-severity vulns unpatched) are in
`KNOWLEDGE-Claude-Craig.md` §8.

**Confirmed (24 Jul 2026)**: all three branches are still live on
`microsoft/coe-starter-kit` and fetchable, despite the repo being archived.
Craig's fork only pulled `main` when it was created, so none of these three
came across — they need to be fetched directly from upstream. Also
re-verified live: all three PRs are still **open** and their `test` +
`license/cla` checks are **green** (an earlier note about "partial check
status" was wrong — the other two check entries are skipped bot workflows,
not failures), and their base commits are only 1–2 commits behind the fork's
current `main` — not meaningfully stale, cherry-picks should apply cleanly.

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

Re-verified 24 Jul 2026: each PR's base commit is only 1–2 commits behind the
fork's current `main` (they're direct ancestors), so drift risk is low
despite the branches themselves being ~2 months old — still worth a quick
confirm rather than assuming:

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

### Step 5 — Security-freshness check → now a separate tracked follow-up

Already run once (24 Jul 2026, `KNOWLEDGE-Claude-Craig.md` §8): the
Dependabot targets don't fully clear known vulns —`brace-expansion` 1.1.14
still leaves a HIGH-severity DoS bug (fixed in 1.1.16), `axios` 0.31.1 still
leaves 12 vulns including several HIGH (fixed floor is 0.33.0 in the 0.x
line, or 1.18.1 latest). Decision: don't fold a bigger version jump into
*this* merge — land the literal Dependabot bumps here, then do the
re-evaluation as its own high-priority item (tracked in `TODO-Craig.md`):

```bash
cd coe-cli && npm outdated && npm audit
```

### Step 6 — Commit with traceability + verify

```bash
git commit -m "Merge dependabot/npm_and_yarn/coe-cli/axios-0.31.1 (was microsoft/coe-starter-kit#11032, unmerged at archive)"
cd coe-cli && npm ci && npm run build && npm test
```

(Confirmed 24 Jul 2026 that `build` + `test` both work locally — see §0 —
so there's no excuse to skip either here even though CI itself doesn't
currently enforce `build`.)

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

### Phase C — publisher/schema (verified 24 Jul 2026 — see `KNOWLEDGE-Claude-Craig.md` §9 for the raw findings)

**Checked all 13 `Solution.xml` files. Three different publishers are
already in play, not one:**

| Publisher (`UniqueName` / display) | Prefix | Used by |
|---|---|---|
| `catteam` / **"Power CAT"** | `cat` | ALM Accelerator, ALM Accelerator for Makers, Pipeline Accelerator, Theming |
| `powerplatformadmin` / "Power Platform Admin" | `admin` | Core Components, Audit/Governance Components, Nurture Components (the biggest ones) |
| `Crc030b` / "CDS Default Publisher" | `cr5cd` | business_value_core |

**Is it tied to Microsoft?** Mixed. "Power CAT" is a literal reference to
Microsoft's internal Customer Advisory Team — a real, direct Microsoft tie.
"Power Platform Admin" is generic/functional, not Microsoft-named, even
though Microsoft wrote it. "CDS Default Publisher" is just an unedited
Dataverse default — an artifact of it never being customized, not a
deliberate brand at all. Neither of the latter two literally says
"Microsoft"; only "Power CAT" does.

**The hard platform constraint**: a component's `CustomizationPrefix`
(`cat_`, `admin_`, `cr5cd_`) is baked into every table/column/flow's logical
name **at creation time** and cannot be renamed afterwards for any existing
component, ever — this is a Dataverse platform rule, not something specific
to this kit, and not something a script or build step can work around.
Retrofitting a new prefix onto existing schema means recreating every table,
column, flow, and app from scratch and migrating any live data — a
ground-up rebuild, not a rebrand. **Stays out of scope**, matching the
original recommendation.

**But that's not actually needed to fix the "looks Microsoft-provided"
concern** — almost nobody sees the prefix (it only shows up to a maker
inspecting schema/advanced find). What people *do* see is the publisher's
**display name** ("Power CAT" in Settings → Solutions) and the outer
solution's branding — and both of those are cheap, safe, and genuinely
buildable:

1. **Cheapest fix (do this regardless)**: rename the publisher
   `<LocalizedNames>`/`<Descriptions>` text directly in the checked-in
   `Solution.xml` files — e.g. `catteam`'s "Power CAT" → "Community CoE
   Starter Kit". Pure text edit to source already under version control, no
   schema impact, no data migration, safe for anyone upgrading in place. The
   existing `deploy-*.yml` pipelines already package and import straight
   from these files, so this is a one-line change that flows through the
   existing build/release process with no new pipeline step needed.
2. **For real brand independence**: create one new umbrella solution +
   publisher (e.g. `communitycoe` / prefix `ccsk`) and add the *existing*
   `admin_`/`cat_`/`cr5cd_`-prefixed tables, flows, and apps into it as
   included components. Dataverse solutions are just containers — a
   component's origin prefix is fixed forever, but which solution "owns" and
   distributes it isn't. This is exactly the pattern used to consolidate the
   8 separate existing solutions under one distributable, cleanly-branded
   package. **Caveat**: existing installs of the original Microsoft-published
   managed solutions can't upgrade onto this new umbrella solution in place
   (new solution `UniqueName` = Dataverse treats it as unrelated) — matches
   what was already flagged, fresh install only for anyone on the old kit.

   **Confirmed (24 Jul 2026) — this is all doable via `pac` CLI, no
   make.powerapps.com needed** (one-time interactive/Entra sign-in via
   `pac auth create` aside — that's login, not the maker portal):

   ```bash
   # 1. Scaffold a new publisher + solution locally
   pac solution init --publisher-name CommunityCoEStarterKit --publisher-prefix ccsk --outputDirectory ./CommunityCoEUmbrella
   cd CommunityCoEUmbrella && dotnet build   # produces an empty solution .zip

   # 2. Import it — this is what actually creates the Publisher + Solution
   #    records live in Dataverse, entirely via CLI
   pac solution import --path ./bin/Debug/CommunityCoEStarterKit.zip --publish-changes

   # 3. Add existing components into it (repeat per component)
   pac solution add-solution-component --solutionUniqueName CommunityCoEStarterKit \
     --component admin_app --componentType 1 --AddRequiredComponents
   ```

   Component type codes are a fixed Microsoft reference list (Table=1,
   Workflow/Flow=29, Web Resource=61, Canvas App=300, ...) — see the
   [SolutionComponent reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/solutioncomponent#BKMK_ComponentType)
   for the full set, including a couple (e.g. AppModule/model-driven apps)
   not on that particular page that are worth double-checking before
   scripting against them.

   **Ponytail call**: `add-solution-component` only takes one component at a
   time by schema name + type code. Scripting that for the ~hundreds of
   pre-existing components across all 8 legacy solutions is real
   effort — for that one-time historical migration, it's genuinely less
   work to do it once via the maker portal's multi-select "Add existing"
   dialog (a few minutes of clicking) than to write and debug a
   componentType-lookup script for a single bulk pass. Reserve the CLI
   command for what it's actually good at: every *new* component created
   from here on, added one at a time as part of the normal build, with
   zero portal use.

**Recommendation**: do (1) as part of Branding Phase A — it directly answers
the "still looks Microsoft" worry, is nearly free, and is the one piece of
this that touches the schema layer at all. Treat (2) as a real but separate
Phase 2 decision (add to `SCOPE-CHARTER.md`'s open-decisions list) — worth
doing before any public release since it's what makes "Community CoE Starter
Kit" the actual publisher of record, but sequence it deliberately rather
than folding it into a routine branding pass. Do not attempt a full
prefix/schema rebuild — that's a separate, much larger project and stays
out of scope per the charter.

---

## 4. Verify/CI the Power Platform solution build (`.cdsproj` → solution `.zip`)

**Why**: `coe-cli`'s build (§0) only covers the Node CLI — it says nothing
about whether the actual CoE Starter Kit deliverables (the Dataverse
solutions) still pack. Confirmed 3 Aug 2026 the mechanism exists and works
locally; it's just unverified in CI. Full findings in
`KNOWLEDGE-Claude-Craig.md` §10.

**Done (3 Aug 2026)**:
1. Built all 11 `.cdsproj` projects locally (the 10 original Microsoft
   solutions + `CommunityCoEUmbrella`) — **11/11 produced valid managed +
   unmanaged solution `.zip` files**, confirmed by checking `bin/Debug/`
   directly rather than trusting the reported exit code alone (see next
   point for why). Full detail in `KNOWLEDGE-Claude-Craig.md` §10.
2. In the process, found and characterized a real, reproducible flake: 6 of
   the 11 builds reported "Build FAILED" on a tight back-to-back run, all
   with the identical `MSB3231: Unable to remove directory
   "obj\Debug\Metadata"` — SolutionPackager's own post-pack cleanup step
   hitting a transient Windows file-lock (almost certainly AV/indexer
   scanning the just-written files), *after* the solution zip had already
   been generated successfully in every case. Added
   `.github/workflows/build-solutions.yml`: a matrix job (one entry per
   `.cdsproj`, `windows-latest`, triggered on PRs touching any solution
   folder). This is the solution-side equivalent of the `coe-cli` build gate
   in §0/§7.
3. **Fixed the MSB3231 race at the root instead of just retrying around it**:
   traced it to the vendor's `CleanUpIntermediateFiles` target having no
   error tolerance on its `RemoveDir`, unlike their own `Clean` target for
   the equivalent case. Added a repo-root `Directory.Build.targets` that
   redefines that one target with the same `ContinueOnError` tolerance —
   MSBuild auto-imports it for every `.cdsproj` under the repo, so this is
   one ~10-line file fixing it for local builds, CI, *and* Visual Studio,
   with no per-project changes needed. Verified by rebuilding all 11 back-
   to-back again: the race still fired (8 of 11 this time) but every one
   downgraded to a warning, and all 11 exited 0. Removed the now-unnecessary
   3-attempt retry from `build-solutions.yml` — a plain `dotnet build` is
   sufficient once the root cause is tolerated correctly.
4. **Reconciled the .NET version gap**: `build-solutions.yml` now pins
   `.NET 10.0.x` (an LTS release — even-numbered .NET versions are LTS),
   matching what's already installed and verified locally. Considered and
   rejected building/testing against multiple .NET versions instead — same
   reasoning as the Node-version decision in §0 item 3: no external consumer
   whose environment we don't control, so one current-LTS version is enough.

**Still open**:
1. Whether a failed solution build should just report (the current bar,
   matching `coe-cli`) or also attempt import into a real Dataverse dev
   environment is a bigger, separate decision — needs environment
   credentials in CI — not attempted here.
2. `ALMAcceleratorForMakers`, `CenterofExcellenceCoreComponentsTeams`, and
   `Theming` are excluded from this workflow entirely (no `.cdsproj`) — see
   §5.

---

## 5. Azure DevOps hosting for the 3 template-only solutions (and the wider ADO/GitHub question)

**Context**: `ALMAcceleratorForMakers`, `CenterofExcellenceCoreComponentsTeams`,
and `Theming` (`KNOWLEDGE-Claude-Craig.md` §10) ship sample Azure DevOps
pipeline YAML instead of a `.cdsproj` — they assume a real ADO project, a
`powercat-alm` GitHub service connection, and an
`alm-accelerator-variable-group` variable group, none of which exist yet.
Craig has corporate Fusion5 ADO access but no repo/project for this there
yet.

**Decision needed** — what to do with these 3 folders (not mutually
exclusive):
- **(a) Leave as reference-only samples** — don't stand up ADO at all,
  accept these 3 solutions have no working local/CI build path (matches
  current state, zero effort).
- **(b) Migrate to the `.cdsproj` pattern** used by the other 11 (§4) — gives
  them the same local/CI build path as everything else, no ADO dependency.
  Mechanically straightforward (same `pac solution init`/unpack shape as
  `CommunityCoEUmbrella` in §3), but loses whatever these 3 pipelines were
  specifically doing via the ALM Accelerator templates (multi-stage
  validation/test/prod promotion) — worth checking what that buys before
  dropping it.
- **(c) Actually stand up the ADO pipelines** — create an ADO project, wire
  the GitHub service connection + variable group, let these 3 run for real
  as ADO-driven CI/CD. Only worth it if there's a genuine reason to want
  ADO-native multi-environment promotion specifically, beyond what the
  `.cdsproj` local-build path already gives you.

**On "store/sync in both ADO and GitHub" (answered 3 Aug 2026, full
reasoning in `KNOWLEDGE-Claude-Craig.md` §10)**: no harm in ADO **pipelines**
reading from the GitHub repo (exactly what the sample templates already
assume, via a service connection) — one canonical repo, ADO as CI engine
only. Real harm is specifically in a second **independently-writable ADO
Repos copy** of the source: sync-conflict risk, plus it blurs the
"personal/public, not Fusion5-controlled" positioning already decided in
Phase 2 (`TODO-Craig.md`). If (c) above is chosen, stand up ADO with
**pipelines + service connection only**, not an ADO Repos mirror.
