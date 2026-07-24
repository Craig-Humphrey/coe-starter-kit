# CoE Starter Kit Fork — Project Memory

Feasibility project for **Community CoE Starter Kit** — a public, independent
(non-Fusion5-branded) community continuation of `microsoft/coe-starter-kit`
(archived 2 Jul 2026). "Fusion5-sponsored" means staff time/leeway, not
funding or branding. Owned/managed solo by Craig for now, with intent to move
to open contributor-led governance later. Currently in Phase 1 (preserve &
assess) — not yet a go decision. See `SCOPE-CHARTER.md` for the full charter.

This repo is worked from **both** Claude Cowork (research, writing, planning)
and Claude Code (technical/repo work). Both read this file automatically at
the start of every session — Cowork inherits Claude Code's CLAUDE.md and
memory system — so this file is the handoff point between them. No copy/paste
between tools should be needed; keep this file and its imports current
instead.

## Always-loaded context

@KNOWLEDGE-Claude-Craig.md
@TODO-Craig.md

## Cross-tool continuity protocol

- Before ending a session in **either** tool: update checkboxes in
  `TODO-Craig.md`, and append any new findings/decisions to
  `KNOWLEDGE-Claude-Craig.md` as a dated entry (`### <date>` near the top of
  the relevant section) rather than rewriting existing content.
- Commit these two files (and this one) at the end of a working session.
  Cowork and Code both read from the same working copy, so a commit is what
  actually carries knowledge from one to the other — an in-chat answer that
  never lands in these files will not be visible next session, in either tool.
- If a session produces a finding worth remembering, it belongs in
  `KNOWLEDGE-Claude-Craig.md`, not just the chat transcript.
- Power Platform ships monthly — re-verify anything time-sensitive in
  `KNOWLEDGE-Claude-Craig.md` before acting on it rather than trusting the
  date it was recorded.

## Rough split of where work happens

- **Cowork**: web research, stakeholder-facing docs/decks, scope charter
  drafting, decision framing.
- **Claude Code**: git operations, merging the pending Dependabot PRs,
  running `coe-cli`, any code/repo restructuring.
Either tool can do either kind of task — this is a default, not a rule.

## Current phase

Phase 1 — Preserve & assess. Live checklist: `TODO-Craig.md`.
