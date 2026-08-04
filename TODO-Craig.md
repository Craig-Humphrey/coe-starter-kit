# TODO — Community CoE Starter Kit: Testing the Waters Cheaply

Goal: get to a real internal decision point on whether Fusion5 gives Craig (and
other Power Platform–focused staff) leeway to sponsor a community fork —
**Community CoE Starter Kit** — for the lowest possible cost/commitment. See
`KNOWLEDGE-Claude-Craig.md` for full background/reasoning, `TODO-Craig-Detailed.md`
for step-by-step execution plans, and `SCOPE-CHARTER.md` for the draft charter.

Everything in Phase 1 is cheap, low-risk, and fully reversible — do these
before spending any real effort on Phase 2+.

## Phase 1 — Preserve & assess

- [x] Fork location — **decided (24 Jul 2026): staying on Craig's personal
      GitHub (`Craig-Humphrey/coe-starter-kit`) for now**, while feasibility
      is assessed. Low risk to defer moving it: a GitHub repo transfer later
      brings issues, PRs, releases, wiki, stars, and existing collaborators'
      access along automatically — see `KNOWLEDGE-Claude-Craig.md`. Revisit
      before any public launch.
- [ ] Export the current Issues backlog from the archived upstream repo —
      full step-by-step plan in `TODO-Craig-Detailed.md` (§1). Confirmed the
      archived repo is still fully readable via the GitHub API/git, so this
      isn't urgent, but do it before relying on that indefinitely.
- [x] **Harden the `coe-cli` build/CI process before merging any dependency
      bumps** — **done (24 Jul 2026)**: CI now runs `npm run build` and
      `npm run lint` as their own steps (previously only `npm test` ran).
      Added a real ESLint config, fixed the script bug that was masking lint
      failures, fixed the handful of resulting trivial errors (`npm run
      lint` now exits 0: 0 errors / 294 tracked warnings). Full details in
      `KNOWLEDGE-Claude-Craig.md` §7. Node version pin — **also done (3 Aug
      2026)**: CI bumped `16.x` (EOL) → `24.x` (current LTS, matches what's
      already installed locally), `actions/setup-node` bumped `@v1` → `@v4`,
      and `coe-cli/package.json` now has `"engines": {"node": ">=20"}`.
      Re-verified the full pipeline still passes clean. **Still open**:
      contributor build docs — tracked in `TODO-Craig-Detailed.md` §0 item 4,
      not blocking the Dependabot merge below.
- [x] **Test guidance + coverage baseline for `coe-cli`** — **done (24 Jul
      2026)**: no test documentation existed anywhere in the repo. Added
      `coe-cli/docs/testing/readme.md` — existing test conventions
      (constructor-injection + jest-mock-extended), a coverage snapshot
      (67.7% stmts overall; `common/cli.ts` 7.4% and `common/prompt.ts` 14.3%
      have zero spec files), and two gap buckets: actionable-now vs.
      genuinely-hard-to-test (the `az` CLI/readline shell-outs with no
      injection seam, and the entire non-`coe-cli` solution layer, which has
      no test framework at all). Linked from `coe-cli/readme.md`.
- [ ] Merge the 3 pending Dependabot branches into the fork — **confirmed
      (24 Jul 2026) these did NOT come across** when the fork was created
      (fork only has `main`); all three are confirmed still live on the
      archived upstream repo and fetchable, still open, and CI-green (see
      `KNOWLEDGE-Claude-Craig.md` §4/§8 — an earlier note about partial CI
      checks was wrong). Sequence **after** the build-hardening item above.
      Full plan in `TODO-Craig-Detailed.md` (§2):
  - [ ] `dependabot/npm_and_yarn/coe-cli/babel/plugin-transform-modules-systemjs-7.29.4` (PR #11039)
  - [ ] `dependabot/npm_and_yarn/coe-cli/brace-expansion-1.1.14` (PR #11036)
  - [ ] `dependabot/npm_and_yarn/coe-cli/axios-0.31.1` (PR #11032)
- [ ] **High priority follow-up** (added 24 Jul 2026): re-run `npm
      audit`/OSV check on the full `coe-cli` dependency tree after the above
      merge lands. Local `npm audit` already found **55 vulnerabilities (14
      low, 14 moderate, 24 high, 3 critical)** across the whole tree — far
      more than the 3 Dependabot-flagged packages, and even the Dependabot
      targets themselves don't fully clear the vulns on `axios`/
      `brace-expansion`. See `KNOWLEDGE-Claude-Craig.md` §8 for the specific
      gaps found.
- [ ] Skim the March 2026 milestone (last active, left ~56% complete) for
      anything that was near-finished and worth closing out first
- [x] **Confirm the Power Platform solution build actually works, and wire
      it into CI** — **done (3 Aug 2026)**: `coe-cli`'s build only covers
      the Node CLI, not the real CoE Starter Kit deliverables. Built all 11
      `.cdsproj` solution projects locally via `dotnet build` — **11/11
      produced valid managed + unmanaged solution zips**. Along the way,
      found and characterized a real, reproducible flake in
      SolutionPackager's own post-pack cleanup step (`MSB3231`, a transient
      Windows file-lock, likely AV/indexer — the zip is always already
      generated before it hits) and added
      `.github/workflows/build-solutions.yml` — the solution-side equivalent
      of the `coe-cli` build gate. **Also fixed the MSB3231 flake at the
      root** rather than just retrying around it: a repo-root
      `Directory.Build.targets` gives the vendor's cleanup step the same
      error tolerance its own `Clean` target already has, auto-applied to
      every solution project with no per-project changes. Verified by
      rebuilding all 11 again — the race still fired (8 of 11) but every one
      downgraded to a warning and all 11 exited 0, so the retry loop in CI
      was removed as unnecessary. Also reconciled the .NET version gap: CI
      now pins `.NET 10.0.x` (LTS, matches what's installed locally). Full
      detail in `KNOWLEDGE-Claude-Craig.md` §10 / `TODO-Craig-Detailed.md`
      §4. **Still open**: a failed solution build only reports today,
      doesn't attempt a real environment import.
- [ ] **Decide what to do with the 3 ADO-template-only solutions**
      (`ALMAcceleratorForMakers`, `CenterofExcellenceCoreComponentsTeams`,
      `Theming`) — they ship sample Azure DevOps pipeline YAML instead of a
      `.cdsproj`, pointing at an external Microsoft template repo and ADO
      resources that don't exist yet. Three options (leave as reference-only
      / migrate to the `.cdsproj` pattern / actually stand up ADO) in
      `TODO-Craig-Detailed.md` §5, not decided yet.
- [x] **ADO + GitHub dual-hosting question — answered (3 Aug 2026)**: no
      harm in Azure DevOps pipelines reading from the GitHub repo via a
      service connection (the intended shape, already assumed by the sample
      templates) — harm is specifically in a second independently-writable
      copy of the source in ADO Repos (sync conflicts, and it blurs the
      "personal/public, not Fusion5-controlled" positioning from Phase 2).
      Recommendation: ADO project with pipelines only, no repo mirror. Full
      reasoning in `KNOWLEDGE-Claude-Craig.md` §10.

## Phase 2 — Scope & governance decisions

*(do before investing more real time — this is the guardrail against the
maintenance-treadmill problem that killed the original)*

- [~] Write a one-page scope charter — **drafted (24 Jul 2026)**, see
      `SCOPE-CHARTER.md`. Still needs your read-through/edits before treating
      it as final.
- [x] Internal vs. public decision — **decided (24 Jul 2026): public
      release**, but **not** under the Fusion5 brand. "Fusion5 sponsoring"
      this means Fusion5 giving Craig and other Power Platform–focused staff
      leeway/time to contribute — no funding, no Fusion5 name or logo
      attached to the project.
- [x] Naming/branding — **decided: "Community CoE Starter Kit"**. Still can't
      use "Microsoft" name/logo or imply MS endorsement (MIT trademark
      condition, unaffected by the Fusion5 question). Full migration plan in
      `TODO-Craig-Detailed.md` (§3).
- [~] Governance model — **partially decided (24 Jul 2026): starts
      solo-owned/managed by Craig**, with explicit intent to move to an open,
      contributor-led model later. **Not yet decided**: the trigger/criteria
      for that handover — needs its own decision, don't let it happen by
      default.
- [x] Repo hosting — **decided (3 Aug 2026): GitHub stays the one canonical
      copy of the source.** Craig has corporate Fusion5 Azure DevOps access
      and no harm in pointing ADO **pipelines** at the GitHub repo via a
      service connection (Fusion5 as CI engine only) — but a second,
      independently-writable copy of the source in ADO Repos would both risk
      sync conflicts and blur the "personal/public, not Fusion5-controlled"
      line decided above. Full reasoning in `KNOWLEDGE-Claude-Craig.md` §10;
      the concrete ADO-pipelines decision for the 3 template-only solutions
      is tracked in Phase 1 above / `TODO-Craig-Detailed.md` §5.
- [ ] Check Microsoft trademark language before any public naming or
      announcement (this is now the only trademark check that matters, since
      the release isn't Fusion5-branded — no separate Fusion5 legal/marketing
      review needed for branding, though see Phase 4 for the separate
      time-allocation ask)

## Phase 3 — Validate external interest (cheap, informal)

- [ ] Informal outreach to Intelogy (UK Power Platform consultancy that
      publicly said they'd like to see a community takeover) — gauge interest
      in co-investment, amplification, or at least a friendly mention
- [x] Keep an eye on the Power Apps Community forum and the archived repo's
      fork network for other emerging community activity — **automated (24
      Jul 2026)**: daily scheduled Claude task now checks both and logs
      anything notable to `KNOWLEDGE-Claude-Craig.md` §3.

## Phase 4 — Internal sign-off

- [ ] Get informal buy-in from Craig's manager/leadership to spend work time
      on this — since there's no Fusion5 $ or brand involved, this is really
      a time-allocation ask (for Craig and any other interested PP staff),
      not a formal legal/governance approval
- [ ] Bring the scope charter + Phase 1 proof-of-work (merged Dependabot PRs,
      exported issue backlog, Branding Phase A done) to whoever else needs to
      be aware this is happening
