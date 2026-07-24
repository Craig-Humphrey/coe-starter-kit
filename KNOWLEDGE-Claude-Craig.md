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
  - All last updated ~2 months before archiving; all showing partial check status (2/4).
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

## 7. Key sources

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
