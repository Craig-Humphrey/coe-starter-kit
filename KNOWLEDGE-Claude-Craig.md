# CoE Starter Kit — Background & Research Notes

Context for a possible Fusion5-sponsored community fork of `microsoft/coe-starter-kit`.
Compiled from research done in Claude chat, July 2026. Treat dates/status as accurate
as of **24 July 2026** — Power Platform and the admin center ship monthly, so re-verify
anything time-sensitive before acting on it.

---

## 1. Deprecation status & timeline

- **Through late 2025**: CoE Starter Kit shipped monthly releases as normal.
- **February 2026**: Monthly releases stopped. Active development ceased — PowerCAT
  team resources were reassigned.
- **7 May 2026**: Microsoft Learn formally updated the starter-kit docs with a
  permanent notice: *"The Power Platform CoE Starter Kit is no longer actively
  maintained. Its core capabilities are part of the Power Platform admin center.
  Issues are no longer reviewed or addressed."* Security vulnerabilities are still
  meant to be routed to MSRC.
- **2 July 2026**: The GitHub repo (`microsoft/coe-starter-kit`) was **archived
  (read-only)**. At archiving: 240 forks, 946 stars.
- The Kit was **never an official Microsoft product** — it was built by the Power
  Customer Advisory Team (Power CAT) and released as a community accelerator. This
  is why there's no formal deprecation notice, no migration deadline, and no
  end-of-support SLA (Microsoft's position, stated via GitHub issue #10990: "was
  never a product").
- Microsoft's recommended path forward: native Power Platform admin center —
  Inventory, Usage, Monitor, and Actions experiences.
- Microsoft's own next-gen answer to the governance gap is **not** community-driven:
  the "Agentic Center of Enablement" (formerly teased at Ignite 2025 as "Virtual
  CoE"), three "guardian agents," preview ~May 2026 / GA ~June 2026 per the 2026
  Release Wave 1 plan.

## 2. Feature / gap comparison — CoE Starter Kit vs. native Admin Center

**Core / Inventory & Admin**

| CoE feature | Native equivalent | Status |
|---|---|---|
| Sync flows → Dataverse inventory tables | Inventory experience (GA) — unified view of apps/flows/agent workflows | ✅ Full |
| Power BI Command Center dashboards | Usage page (public preview) | ✅ Full (preview) |
| DLP Editor (edit policies in-app) | Native Security > Data and privacy > Data policy | ✅ Full (was always native) |
| DLP Impact Analysis (test policy before publishing) | Advanced Connector Policies (GA June 2026) + inventory connector/operation usage preview | 🟡 Partial — foundation landing, not full parity |
| Set App Permissions / Admin – App Permission Center (transfer app ownership via UI) | None for canvas apps — `Set-AdminPowerAppOwner` PowerShell or export/reimport only | ❌ Gap |
| Change flow owner | Native UI, GA since Feb 2022 | ✅ Full (flows only) |

**Governance / Compliance**

| CoE feature | Native equivalent | Status |
|---|---|---|
| Developer Compliance Center + Compliance Detail Request | None | ❌ Gap |
| App/Flow/Bot/Custom Connector Approval BPFs | None | ❌ Gap |
| App quarantine process | None (Advisor recommends, doesn't auto-quarantine) | ❌ Gap |
| Inactivity notifications (owner-approval-then-delete workflow) | Partial — Advisor flags low usage, no full workflow | 🟡 Partial |
| Cleanup for orphaned resources (departed-employee apps/flows → manager reassignment) | None | ❌ Gap |
| Teams environment governance (business justification + cleanup) | Partial — auto-delete of inactive Dataverse-for-Teams environments is native; justification-request flow isn't | 🟡 Partial |

**Nurture / Community**

| CoE feature | Native equivalent | Status |
|---|---|---|
| Video hub, Training in a Day, Maker assessment, Pulse survey, Newsletter, Innovation Backlog | None | ❌ Gap (all) |
| Template Catalog | Partial — Managed Environments "maker welcome content" is a much thinner substitute | 🟡 Partial |

**Net read:** visibility/reporting is now genuinely well covered natively. The real
gaps are all **process**: compliance workflows, quarantine, orphaned-resource
reassignment, and maker nurture/community features. That's where a fork would
actually add value — not in rebuilding inventory or dashboards.

## 3. Community succession landscape (as of July 2026)

- **No confirmed active community-led successor project found.** What exists is the
  usual long tail of personal/org forks (e.g. individual GitHub accounts with their
  own copy) — standard "forked it to customize for our tenant" copies, not
  community governance projects with their own release cadence or roadmap.
- Sentiment exists but no action yet: Intelogy (UK M365/Power Platform
  consultancy) publicly wrote that it "remains to be seen" whether the community
  will step in, and said they'd like to see it happen — i.e. hope, not a project.
- **Relevant precedent**: *xRM Portals Community Edition* — Adoxio's fork of
  Microsoft's deprecated Dynamics 365 Customer Engagement Portals, which became the
  recognized continuation path for customers upgrading off the old Adxstudio
  Portals product. Adoxio is the same kind of Microsoft-ecosystem consulting
  partner Fusion5 is — good direct comparable for how naming/governance/support
  boundaries were handled.

## 4. Repo state at archiving — branches & issues

### 24 July 2026 — verified against live GitHub data
- All three pending Dependabot branches are **confirmed still present and
  fetchable** on the archived `microsoft/coe-starter-kit` repo (checked live
  via the GitHub REST API): `dependabot/npm_and_yarn/coe-cli/brace-expansion-1.1.14`
  (commit `382a958c`), `dependabot/npm_and_yarn/coe-cli/axios-0.31.1` (commit
  `66c0a1cf`), and `dependabot/npm_and_yarn/coe-cli/babel/plugin-transform-modules-systemjs-7.29.4`
  (PR #11039, still shown as `open`). Archiving does **not** delete branches
  or block reads — git clone/fetch and the REST/GraphQL APIs remain fully
  functional against a public archived repo; it only blocks new pushes,
  issues, and PR/issue comments through GitHub itself (all threads on the
  archived repo are now shown as **locked**).
- Craig's fork (`Craig-Humphrey/coe-starter-kit`) currently has **only
  `main`** — confirmed the Dependabot branches did not come across when the
  fork was created. They'll need to be pulled directly from upstream — full
  plan in `TODO-Craig-Detailed.md` §2.

- **"Active branches" at archive time = 3 Dependabot branches only**, all against
  the peripheral `coe-cli` Node tool, none merged:
  - `dependabot/npm_and_yarn/coe-cli/babel/plugin-transform-modules-systemjs-7.29.4` (PR #11039)
  - `dependabot/npm_and_yarn/coe-cli/brace-expansion-1.1.14` (PR #11036)
  - `dependabot/npm_and_yarn/coe-cli/axios-0.31.1` (PR #11032)
  - All last updated ~2 months before archiving. **Correction (24 Jul 2026,
    re-verified live)**: the original "partial check status (2/4)" note was
    wrong — all three PRs' `test` and `license/cla` checks report `success`;
    the other two entries are bot workflows (`issue_opened_or_reopened`,
    `issue_closed`) that report `skipped`, not failing. All three are green.
    Also: their base commits (`e5b8162`, `4a4d5463`) are direct ancestors of
    the fork's current `main` — only 1–2 commits behind, not meaningfully
    stale. Cherry-picks should apply cleanly. Full re-assessment in §8.
- **The real bug backlog was never in branches — it's in Issues**, and was never
  cleared: known unresolved bugs at end-of-life included stale object alerts firing
  for deleted flows, inventory sync failures, BYODL/Dataflow failures in GCC
  environments, and the inactivity-cleanup flow failing to enable.
- The **March 2026 milestone** — one of the last active release cycles — was left
  at **56% complete (34 open / 44 closed)**, overdue by a month when work stopped.
- **Conclusion**: this was not a clean wind-down. Development stopped mid-flight
  (in-progress milestone, unmerged routine maintenance, open bugs against recent
  releases), not at a deliberate stopping point.
- Note for forking: **GitHub Issues/PR discussion threads do not transfer with a
  fork** (they're platform data, not git data). If the backlog is worth preserving
  as a reference, it needs a manual export/scrape while the archived repo is still
  readable — don't assume it'll still be reachable indefinitely.

## 5. Legal / IP position

- **License: MIT** (Copyright Microsoft Corporation). Permits use, copy,
  modification, merger, publishing, distribution, sublicensing, and sale — the only
  condition is retaining the copyright/permission notice. This is about as
  permissive as it gets; no need to ask Microsoft's permission to fork and
  continue development.
- **Trademark constraint**: per the repo's own guidance, use of Microsoft
  trademarks/logos in modified versions "must not cause confusion or imply
  Microsoft sponsorship." Practical effect: a fork needs **its own name** — can't
  be shipped as "Microsoft CoE Starter Kit, maintained by Fusion5."
- The original **CLA** (Contributor License Agreement) applied only to
  contributions flowing back into Microsoft's repo — it doesn't carry forward. A
  new project needs its own contributor agreement if it wants to accept external
  PRs with clear IP terms.

## 6. Strategic considerations for a Fusion5-sponsored fork

### 24 July 2026 — decisions locked in
- **Name**: *Community CoE Starter Kit*.
- **Release type**: public, but **not** under the Fusion5 brand/logo.
  "Fusion5 might sponsor it" means Fusion5 giving Craig — and other Power
  Platform–focused Fusion5 staff — leeway/time to contribute, not funding and
  not Fusion5's name attached to the project. This removes most of the
  Fusion5-trademark exposure discussed below; the Microsoft-trademark
  constraint (can't imply MS endorsement) still applies regardless of who's
  contributing time.
- **Ownership/governance**: starts solo — owned and managed by Craig
  personally. Explicit intent to move to an open, contributor-led governance
  model later; the trigger/criteria for that handover isn't decided yet (see
  `TODO-Craig.md` Phase 2).
- **Repo location**: staying on Craig's personal GitHub
  (`Craig-Humphrey/coe-starter-kit`) while feasibility is assessed, not yet
  moved to a Fusion5-controlled org. Confirmed this is low-risk to defer:
  GitHub repository transfers bring issues, PRs, releases, wiki, stars,
  watchers, webhooks, secrets, and existing collaborators' access along
  automatically (GitHub Docs, "Transferring a repository"). The only real
  constraint on a personal account is a flatter permission model (no Teams)
  and, for **private** repos only, a 3-collaborator cap on the Free plan —
  moot here since this is a public repo.

- **Biggest risk**: recreating the exact maintenance treadmill that ended the
  original. Microsoft's own PowerCAT team had dedicated headcount and still
  shelved it when priorities shifted — a fork with less dedicated capacity won't
  automatically fare better against the same pace of platform change. Tight scope
  is the mitigation, not optional:
  - Bug fixes only (no new features)
  - Prune/retire components now redundant with the native Admin Center (see gap
    table above — anything marked ✅ Full is a candidate for removal, not upkeep)
  - No org/client-specific customization in the shared fork
- **Elevated permission surface**: several components run with real reach —
  tenant-wide Dataverse read access, app ownership reassignment, app quarantine.
  Once Fusion5's name is attached to a tool with that scope, "we'll fix bugs when
  we get to it" carries different weight than it does for a hobby fork. Needs an
  explicit decision, not a default:
  - Internal/client-engagement asset only, **vs.**
  - Public release under the Fusion5 brand, open to anyone
  — these carry genuinely different security-response and support commitments.
- **Competitive context**: Rencore already sells a commercial Power Platform
  governance product covering overlapping ground. Worth being intentional about
  positioning (goodwill/thought-leadership play vs. undercutting a paid-tool
  market) rather than backing into it.
- **Governance model decision**: Fusion5-solo-maintained (simpler, full control)
  vs. genuine open multi-org community project (harder to bootstrap, needs
  contributor agreements, more credible/sustainable long-term if it actually
  attracts outside contributors). "Sponsor a community fork" implies leaning
  toward the latter — worth naming that choice explicitly rather than assuming it.

## 7. Build process & dependency health (`coe-cli`) — verified 24 Jul 2026

Findings from actually running the build locally before merging any Dependabot
PRs, prompted by the realization that "does it build" wasn't verified anywhere.

**Documentation coverage — thin.** `HOW_TO_CONTRIBUTE.md`/`CONTRIBUTING.md`
cover git workflow and CLA, not build steps. `coe-cli/readme.md` documents
*end-user* install requirements (Node 12+, Azure CLI) for running the compiled
CLI, not contributor build/test steps. `coe-cli/docs/cli-development/readme.md`
is a dead stub pointing at Microsoft Learn. No document anywhere says "run
`npm install && npm run build && npm test`" — the only executable source of
truth is the CI workflow.

**CI coverage — incomplete.** `.github/workflows/pr-loop-jest-tests.yml` is
the only build-related workflow: pins **Node 16.x** on `windows-latest`, runs
`npm install` + `npm test` in `/coe-cli`. It does **not** run `npm run build`
(tsc) or `npm run lint` — a change that breaks compilation could still pass
CI today. `coe-cli/package.json` has no `engines` field, so nothing pins a
Node version at the package level either. (Other workflows —
`assign-issue-to-project.yml`, `auto-reply`, `sync-to-azdo.yml` — are unrelated
to build.)

**Locally verified working**, using the tools already installed (Node
v24.15.0, npm 11.12.1 — no separate setup needed):
- `npm install` — succeeds. 763 packages, one `EBADENGINE` warning
  (`@azure/msal-node@1.14.6` wants Node 10/12/14/16/18), and **55 known
  vulnerabilities across the full tree (14 low, 14 moderate, 24 high, 3
  critical)** per `npm audit` — this is much broader than just the 3
  Dependabot-flagged packages. Feeds the "high-priority npm-audit
  re-evaluation" follow-up task in `TODO-Craig.md`.
- `npm run build` (`tsc`) — clean, exit 0, despite the pinned TypeScript
  being `^4.5.5` (2022-era) running under Node 24.
- `npm test` (jest) — **12 suites / 89 tests, all pass**.
- `npm run lint` (eslint) — **broken**. No `.eslintrc*` has ever existed in
  `coe-cli`'s git history, so ESLint has nothing to load and errors out. The
  script itself (`eslint . & echo 'lint complete'`) uses `&` not `&&`, so the
  npm script always exits 0 regardless — this has likely silently no-op'd for
  the project's entire life, not something recent drift broke.

**Net read**: the actual build is healthy (compiles, tests pass) — the gap is
process, not code: no documented contributor build steps, no Node version
pin, no build/lint step in CI, and a lint config that's never existed. Worth
fixing before merging dependency bumps so a broken bump would actually be
caught.

**Fixed (24 Jul 2026, same day)**: CI (`pr-loop-jest-tests.yml`) now runs
`npm run build` and `npm run lint` as separate steps, so either failing now
fails CI. Added `coe-cli/.eslintrc.json` (TypeScript-recommended rules, with
the purely-mechanical legacy-style ones — `no-var`, `prefer-const`,
`no-var-requires`, `no-empty-function` — turned off since 962 of the initial
1008 errors were exactly those and 100% pre-existing debt unrelated to
correctness). Fixed the `lint` script's `&`-masking bug. Ran `eslint --fix`
for the handful of remaining trivial, auto-fixable production-code issues;
the one non-auto-fixable case got a scoped disable comment rather than a
behavioral change. `npm run lint` now genuinely exits 0 (0 errors, 294
warnings — tracked debt, mostly `no-explicit-any`/`no-unused-vars`, not
blocking). `eslint` itself added as an explicit devDependency (was only
resolving transitively before). Remaining open items (Node version/`engines`
pin, contributor build docs) are tracked in `TODO-Craig-Detailed.md` §0.

**Test coverage baseline** (`npx jest --coverage`, 24 Jul 2026): 67.7%
statements / 48.9% branches / 58.9% functions / 68.5% lines overall. Weakest
spots: `src/common/cli.ts` (7.4% stmts), `src/common/prompt.ts` (14.3%),
`src/commands/ebook.ts` (39.5%), `src/common/environment.ts` (40.2%),
`src/commands/login.ts` (50%). All 10 `src/commands/*.ts` files have a
matching `test/commands/*.spec.ts`; `src/common/` has specs for only
`config.ts` and `environment.ts` (`cli.ts`, `prompt.ts`,
`readLineManagement.ts` don't). No test coverage exists anywhere outside
`coe-cli` — the Dataverse solution folders (`CenterofExcellence*`,
`ALMAccelerator*`, etc., the bulk of the actual CoE Starter Kit) have no
automated test framework at all.

## 8. Dependabot branch re-assessment (24 Jul 2026)

Before merging, checked live PR state via the GitHub REST API and cross-
referenced each pinned version against the [OSV database](https://osv.dev)
(`api.osv.dev`) rather than trusting the ~2-month-old Dependabot suggestions
verbatim.

| Dep | Current (in fork `main`) | Dependabot target | OSV result |
|---|---|---|---|
| `@babel/plugin-transform-modules-systemjs` | 7.16.7 — 1 HIGH vuln | 7.29.4 | **Fully fixes it** — 0 vulns remain |
| `brace-expansion` (transitive) | 1.1.11 — 3 vulns (1 HIGH) | 1.1.14 | Fixes 2 of 3; **leaves the HIGH-severity DoS bug unpatched** (`GHSA-3jxr-9vmj-r5cp`, fixed only in 1.1.16) |
| `axios` (direct dep) | 0.25.0 — **23 known vulns** | 0.31.1 | Still leaves **12 vulns, several HIGH** (Proxy-Authorization credential leak on redirect, ReDoS via cookie injection, NO_PROXY/SSRF bypasses). Current npm 0.x floor is 0.33.0; latest overall is 1.18.1. |

`axios` usage in `coe-cli` is trivial (`axios.get<string>(url, config).data`
in `src/commands/devops.ts`), so a bigger version jump carries low
breaking-API risk whenever that's tackled.

**Decision (24 Jul 2026)**: don't try to leapfrog straight to fully-patched
versions in the same pass as bringing in the stale Dependabot branches — do
the build-process hardening (§7) first, then merge the 3 Dependabot PRs as
originally planned (`TODO-Craig-Detailed.md` §2, cherry-pick approach), then
run a dedicated `npm audit`/dependency-floor re-evaluation as a high-priority
follow-up (tracked in `TODO-Craig.md`) rather than bundling a bigger version
jump into the "bring in what Dependabot already proposed" step.

## 9. Publisher/schema branding — verified 24 Jul 2026

Checked all 13 `Solution.xml` files (`**/Other/Solution.xml`) for
`<Publisher>` and `CustomizationPrefix`. Three different publishers are
already in use across the kit, not one:

| Publisher `UniqueName` / display name | `CustomizationPrefix` | Used by |
|---|---|---|
| `catteam` / **"Power CAT"** | `cat` | `CenterofExcellenceALMAccelerator`, `ALMAcceleratorForMakers`, `CenterofExcellencePipelineAccelerator`, `Theming` |
| `powerplatformadmin` / "Power Platform Admin" | `admin` | `CenterofExcellenceCoreComponents`, `CenterofExcellenceAuditComponents`, `CenterofExcellenceNurtureComponents` (the largest/most-used solutions) |
| `Crc030b` / "CDS Default Publisher" | `cr5cd` | `business_value_core` |

(Not yet checked individually: `ALMAcceleratorSampleSolution`,
`CenterofExcellenceAuditLogs`, `CenterofExcellenceCoreComponentsTeams`,
`CenterofExcellenceInnovationBacklog`, `admintaskanalysis_core` — likely one
of the three above given the pattern, confirm before relying on it.)

**Take**: none of these literally say "Microsoft." "Power CAT" is a direct
reference to Microsoft's internal Customer Advisory Team — the one genuine
Microsoft tie. "Power Platform Admin" is generic/functional. "CDS Default
Publisher" is just an unedited Dataverse default, not a deliberate brand.

**Hard constraint**: `CustomizationPrefix` is baked into every existing
table/column/flow's logical name at creation and cannot be renamed for
existing components — a Dataverse platform rule, not fixable by any script
or build step. Full plan (what's cheap vs. what's a separate rebuild
project) is in `TODO-Craig-Detailed.md` §3, Phase C.

## 10. Power Platform solution build process — verified 3 Aug 2026

Investigated what actually produces the CoE Starter Kit's real deliverables
(the Dataverse solution `.zip` files — flows, canvas apps, tables, etc.
combined into one package) since the `coe-cli` build/CI work in §7 only
covers the peripheral Node tool, not the solutions themselves.

**Inventory — three different build shapes across the 14 solution folders:**
- **11 folders have a `.cdsproj`** (the standard
  `Microsoft.PowerApps.MSBuild.Solution` MSBuild project type):
  `admintaskanalysis_core`, `ALMAcceleratorSampleSolution`,
  `business_value_core`, `CenterofExcellenceALMAccelerator`,
  `CenterofExcellenceAuditComponents`, `CenterofExcellenceAuditLogs`,
  `CenterofExcellenceCoreComponents`, `CenterofExcellenceInnovationBacklog`,
  `CenterofExcellenceNurtureComponents`, `CenterofExcellencePipelineAccelerator`,
  and the new `CommunityCoEUmbrella` (§9). `dotnet build` on any of these
  invokes SolutionPackager and produces both managed + unmanaged solution
  `.zip` files under `bin/Debug/`.
- **3 folders have no `.cdsproj` at all** — `ALMAcceleratorForMakers`,
  `CenterofExcellenceCoreComponentsTeams`, `Theming`. Instead each carries
  `deploy-{validation,test,prod}-*.yml`: sample Azure DevOps pipeline
  templates referencing an *external* repo
  (`Microsoft/coe-alm-accelerator-templates`) via `resources.repositories`
  (type: github, endpoint: `powercat-alm` service connection) plus an ADO
  variable group (`alm-accelerator-variable-group`). None of that exists in
  this repo/account — these are reference templates, not runnable pipelines,
  and there's no local/CI way to build these 3 today.
- **`export-unpack.ps1`** (repo root) is the reverse direction: `pac solution
  export` (pull from a live Dataverse env) → `pac solution unpack` (write
  into source). A dev-inner-loop helper, not a build step — needs a live
  authenticated environment + `pac` CLI + `jq`.

**Verified locally (3 Aug 2026)**: built `CenterofExcellenceAuditLogs`
(smallest solution, 13 files) via `dotnet build` in
`CenterofExcellenceAuditLogs/SolutionPackage/` — confirms the toolchain works
end-to-end (.NET 10 SDK already installed, NuGet restore of
`Microsoft.PowerApps.MSBuild.Solution` 1.52.1 succeeds, SolutionPackager
produces both `CenterofExcellenceAuditLogs.zip` and `_managed.zip`). First
attempt reported "Build FAILED" but only on a post-build cleanup step
(`MSB3231: Unable to remove directory obj\Debug\Metadata` — a transient
Windows file-lock, likely AV/indexer/OneDrive holding a handle open right
after the zip was written); the zips were already generated successfully
before that error. Immediate retry succeeded cleanly, 0 errors. **Not yet
tried** on the other 10 `.cdsproj` projects — no reason to expect a different
result, but unverified.

**Gap**: none of these 11 `.cdsproj` builds are wired into any CI workflow —
nothing automatically confirms the actual solution deliverables still pack
after a change, unlike `coe-cli` now (§7). Tracked in `TODO-Craig.md` /
`TODO-Craig-Detailed.md` §4.

**Update (3 Aug 2026, later same day) — all 11 built, and a CI workflow now
covers them:**

Ran `dotnet build` against all 11 `.cdsproj` projects (the 10 original
Microsoft solutions + `CommunityCoEUmbrella`). **Result: 11/11 produced valid
managed + unmanaged solution `.zip` files** — confirmed by checking `bin/Debug/`
directly, not just the reported exit code (see why below).

**Found and characterized a real, reproducible flake**: 6 of the 11 builds
reported "Build FAILED" on the first pass when run back-to-back in a tight
loop, all with the *exact same* error — `MSB3231: Unable to remove directory
"obj\Debug\Metadata"` — SolutionPackager's own post-pack cleanup step hitting
a Windows file-lock on files it just wrote (almost certainly antivirus
real-time scanning or the search indexer briefly holding a handle open). In
**every single one** of those 6 cases, the log confirms `Solution Package
Type: Both generated` appeared *before* the cleanup error — the actual
packing always succeeded; only the temp-folder cleanup afterward flaked.
Retrying immediately didn't always clear it in a tight sequential loop
(`CenterofExcellenceAuditLogs` failed twice in a row, having built cleanly on
its own minutes earlier) — this points at contention from running many
builds back-to-back on one machine, not a per-project defect.

**Added `.github/workflows/build-solutions.yml`**: a matrix job (one entry
per `.cdsproj`, `windows-latest` — required, since these target `net462`)
that runs `dotnet build` per solution with a 3-attempt retry specifically to
absorb the MSB3231 cleanup race, triggered on PRs touching any of the 11
solution folders. This is the solution-side equivalent of the `coe-cli`
build gate in §7 — closes the gap called out above. Uses `.NET 8.0.x` (LTS)
in CI; local verification was done on the already-installed .NET 10 SDK —
that's an unreconciled version gap, same shape as the Node-version pin gap
already tracked for `coe-cli` (`TODO-Craig-Detailed.md` §0 item 3), not
assumed to be equivalent.

Excluded from both the manual build pass and the new workflow:
`ALMAcceleratorForMakers`, `CenterofExcellenceCoreComponentsTeams`,
`Theming` — no `.cdsproj`, ADO-template-only, per the inventory above.

**Azure DevOps + GitHub hosting — considered 3 Aug 2026**: Craig has
corporate Fusion5 Azure DevOps access but no ADO repo for this project yet,
and asked whether storing/syncing this in both ADO and GitHub causes harm.
Two different things get conflated under "sync":
- **ADO pipelines pulling from the GitHub repo** (exactly what the sample
  `deploy-*.yml` templates already assume, via a GitHub service connection)
  — no harm, this is the intended shape: one canonical repo (GitHub), ADO
  used purely as the pipeline/CI engine that reads it. No second copy of
  source to keep in sync.
- **A second, independently-writable copy of the source living in ADO
  Repos** — real harm on two fronts: (a) mechanical — dual-primary sync
  conflicts, divergent history, no clear source of truth; (b)
  governance/branding — this project is deliberately positioned as
  personal/public and explicitly *not* Fusion5-branded or funded (§6,
  `TODO-Craig.md` Phase 2). Hosting the actual source in a Fusion5-controlled
  corporate ADO org, even unofficially, blurs that line in a way a read-only
  pipeline connection doesn't.

**Recommendation**: stand up an ADO **project with pipelines only** — no ADO
Repos copy — pointed back at the GitHub repo via a service connection,
matching what the existing `deploy-*.yml` templates already expect. Revisit
only if there's a specific corporate-compliance reason the source itself
must live in ADO Repos, and treat that as its own explicit decision rather
than a default (same pattern as the publisher/prefix decision in §9).

## 11. Key sources

- Microsoft Learn — CoE Starter Kit transition notice:
  https://learn.microsoft.com/en-us/power-platform/guidance/coe/starter-kit
- Microsoft Learn — governance components (Compliance Detail Request, quarantine,
  orphaned-resource cleanup, Teams governance):
  https://learn.microsoft.com/en-us/power-platform/guidance/coe/governance-components
- Microsoft Learn — nurture components (video hub, Training in a Day, maker
  assessment, pulse survey, newsletter):
  https://learn.microsoft.com/en-us/power-platform/guidance/coe/nurture-components
- Microsoft Learn — core components (sync flows / Dataverse tables):
  https://learn.microsoft.com/en-us/power-platform/guidance/coe/core-components
- GitHub — `microsoft/coe-starter-kit` (archived 2 Jul 2026):
  https://github.com/microsoft/coe-starter-kit
- GitHub — LICENSE (MIT): https://github.com/microsoft/coe-starter-kit/blob/main/LICENSE
- GitHub — HOW_TO_CONTRIBUTE.md (branching model, CLA):
  https://github.com/microsoft/coe-starter-kit/blob/main/HOW_TO_CONTRIBUTE.md
- Microsoft Power Platform Blog — Advanced Connector Policies GA (June 2026):
  https://www.microsoft.com/en-us/power-platform/blog/2026/06/04/advanced-connector-policies-are-generally-available/
- Microsoft Power Platform Blog — March 2026 feature update (Inventory GA, Usage
  preview): https://www.microsoft.com/en-us/power-platform/blog/power-apps/whats-new-in-power-platform-march-2026-feature-update/
- Intelogy blog — "Microsoft ends support for Power Platform CoE" (community
  takeover sentiment, PowerCAT context, Managed Environments/Environment Groups
  history): https://www.intelogy.co.uk/blog/microsoft-is-no-longer-maintaining-the-power-platform-center-of-excellence-starter-kit/
- Rencore blog — "Power Platform CoE Starter Kit: End of Life" (timeline,
  competing commercial governance product):
  https://rencore.com/en/blog/power-platform-coe-starter-kit-end-of-life
- "Life after the CoE Starter Kit" (Perspectives Plus) — Agentic Center of
  Enablement / Virtual CoE background: https://www.perspectives.plus/p/life-after-the-coe-starter-kit
- xRM Portals Community Edition (precedent for partner-led fork continuation):
  https://github.com/manojattal/xRM-Portals-Community-Edition
