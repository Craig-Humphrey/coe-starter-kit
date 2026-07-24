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
- [ ] Merge the 3 pending Dependabot branches into the fork — **confirmed
      (24 Jul 2026) these did NOT come across** when the fork was created
      (fork only has `main`); all three are confirmed still live on the
      archived upstream repo and fetchable. Full plan in
      `TODO-Craig-Detailed.md` (§2):
  - [ ] `dependabot/npm_and_yarn/coe-cli/babel/plugin-transform-modules-systemjs-7.29.4` (PR #11039)
  - [ ] `dependabot/npm_and_yarn/coe-cli/brace-expansion-1.1.14` (PR #11036)
  - [ ] `dependabot/npm_and_yarn/coe-cli/axios-0.31.1` (PR #11032)
- [ ] Skim the March 2026 milestone (last active, left ~56% complete) for
      anything that was near-finished and worth closing out first

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
