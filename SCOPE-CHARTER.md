# Scope Charter — Community CoE Starter Kit

**Status: Draft v1 — 24 July 2026 — not yet finalized or signed off.**

## Problem statement

Microsoft's Power Platform CoE Starter Kit was archived 2 July 2026, with no
official successor beyond native Admin Center features. Those native
features now cover visibility/reporting well, but leave real gaps in
governance/compliance process (approval workflows, quarantine, orphaned-
resource reassignment) and maker nurture features. No confirmed active
community successor exists yet. This charter proposes a scoped, community-run
continuation: **Community CoE Starter Kit**.

## In scope

- Bug fixes and maintenance only for existing CoE Starter Kit components.
- Pruning components now fully covered by the native Admin Center, rather
  than maintaining redundant surface area.
- Preserving the existing Issues backlog and the three pending Dependabot
  updates as the starting baseline.
- Re-branding away from Microsoft naming/trademarks at the documentation and
  repository level.

## Explicitly out of scope (for now)

- New features beyond parity with the archived project.
- Org- or client-specific customization inside the shared/public codebase.
- Renaming the underlying Dataverse solution publisher/schema prefix —
  deferred; see Open Decisions below.

## Release model

- Public repository, MIT-licensed (inherited from the original), openly
  available to anyone.
- **Not** released under the Fusion5 brand or logo — no implication of
  Fusion5 ownership or endorsement.
- Not a Microsoft product; must not imply Microsoft sponsorship or
  endorsement (a trademark condition carried over from the original MIT
  license).

## What "Fusion5 sponsorship" actually means

- No funding, no brand attachment.
- Fusion5 giving Craig — and other Power Platform–focused Fusion5 staff who
  want to contribute — leeway/time to work on it as part of their role.
- Still needs informal buy-in from Craig's manager/leadership as a
  time-allocation ask, separate from any legal or trademark process, since
  there's no brand or IP exposure to Fusion5 itself.

## Ownership & governance

- **Now**: owned and solely maintained by Craig Humphrey, on his personal
  GitHub account.
- **Target**: transition to an open, contributor-led governance model
  (multiple maintainers, a clearer contribution process) once the project
  has shown enough traction/interest to justify it.
- **Not yet decided**: the specific trigger or criteria for that handover —
  flagged as an open decision rather than left to happen by default.

## Name

**Community CoE Starter Kit.**

## Decision point — Phase 1 → Phase 2

Move to the next phase once:

- The Issues backlog has been exported from the archived upstream repo.
- The three pending Dependabot updates have been merged.
- Branding Phase A (README, repo name/description, NOTICE) is complete.

At that point, bring this charter plus the Phase 1 proof-of-work to whoever
needs to be aware this is happening (Phase 4 of `TODO-Craig.md`).

## Open decisions still pending (not blocking Phase 1)

- Governance transition trigger/criteria.
- Whether/when to rebrand the Dataverse publisher/schema prefix (breaking
  change for existing installs vs. clean brand independence).
- Whether/when to move the repo off Craig's personal GitHub to a neutral or
  Fusion5-hosted-but-unbranded location.
