# Testing

This document didn't exist before 24 Jul 2026 — there was no written guidance on
how `coe-cli` is tested, and no record of coverage or gaps. This captures both,
based on reading the existing test suite rather than any prior spec.

Scope note: this only covers `coe-cli`, the Node CLI tool. The rest of the
repository — the actual CoE Starter Kit solution folders (`CenterofExcellence*`,
`ALMAccelerator*`, etc.: Power Automate flows, canvas/model-driven apps, Power BI
reports) — has no automated test framework at all. See "Genuinely hard to test"
below.

## How the existing tests work

- **Framework**: Jest ^27, run via `npm test`. Source is TypeScript, transformed
  by `babel-jest` (see `jest` config in `coe-cli/package.json`) — tests run
  directly against `src/`, not the `built/` output, so `npm test` doesn't
  require `npm run build` first.
- **Location/naming**: `test/` mirrors `src/` — `src/commands/devops.ts` →
  `test/commands/devops.spec.ts`. One spec file per source file.
- **Two established patterns, pick whichever fits**:
  1. **Pure-function table tests** — e.g. [`test/common/environment.spec.ts`](../../test/common/environment.spec.ts):
     plain `describe`/`test`, no mocks, one `test()` per input/output case, one
     assertion each. Use this for logic with no external dependency.
  2. **Mocked-dependency tests** — e.g. [`test/commands/devops.spec.ts`](../../test/commands/devops.spec.ts):
     uses [`jest-mock-extended`](https://www.npmjs.com/package/jest-mock-extended)'s
     `mock<T>()` to stub SDK interfaces (Azure DevOps SDK, Octokit, MSAL,
     `HttpClient`), driven through the constructor-injection convention below.
- **Constructor-injection convention**: command classes (`DevOpsCommand`,
  `LoginCommand`, `EbookCommand`, etc.) accept their I/O dependencies — `fs`
  functions, an HTTP client, a `createWebApi`/`createClientApp` factory,
  `runCommand` — as constructor parameters, defaulting to the real
  implementation but overridable in tests. **This is what makes the existing
  code testable at all** — new command code should follow it rather than
  calling `fs`/a SDK client/`child_process` directly.
- **No integration/E2E tier** — every test is a mocked unit test. Nothing in
  `coe-cli`'s test suite calls a real Dataverse environment, DevOps org, or
  Azure AD tenant.
- **No coverage gate** — `jest` config has no `coverageThreshold`, and CI
  doesn't run with `--coverage`. Coverage can regress silently.

## Coverage snapshot (`npx jest --coverage`, 24 Jul 2026)

Overall: **67.7% statements / 48.9% branches / 58.9% functions / 68.5% lines**.

| Area | Stmts | Has a spec file? |
|---|---|---|
| `src/commands/*.ts` (10 files) | 39–89%, varies per file | Yes, all 10 |
| `src/common/config.ts` | 62.7% | Yes |
| `src/common/environment.ts` | 40.2% | Yes (many branches untested) |
| `src/common/readLineManagement.ts` | 80% | No — incidentally covered via other tests |
| `src/common/cli.ts` | **7.4%** | **No** |
| `src/common/prompt.ts` | **14.3%** | **No** |

Weakest command files: `ebook.ts` (39.5%), `login.ts` (50%).

## Gaps — actionable now (testable today, just not written)

- `src/common/cli.ts` and `src/common/prompt.ts` have no dedicated spec file
  at all (see caveat below — `cli.ts` needs a small refactor first).
- `ebook.ts`'s untested branches are plain fs/string logic behind the same
  constructor-injection pattern already used elsewhere (see its `defaultFs`
  constructor param) — no blocker, just not written yet.
- `login.ts`'s MSAL device-code flow is injectable via `createClientApp`, the
  same way `devops.ts`/`powerplatform.ts` inject their API clients — same
  pattern, just needs the same treatment applied.
- No `coverageThreshold` in the `jest` config and no `--coverage` in CI, so
  there's no signal if coverage drops on a future change.

## Gaps — genuinely hard to test (bookmarked for a rainy day)

- **`common/cli.ts`** (`runCommand`, `validateAzCliReady`) shells out directly
  to the real `az` CLI via `child_process.exec` with **no injection seam** —
  unlike every other command class. Testing it as-is means either mocking the
  `child_process` module wholesale (invasive, brittle) or refactoring it first
  to accept an injectable exec function, matching the convention everywhere
  else. That refactor is the actual prerequisite, not just "write a spec."
- **`common/prompt.ts`** (`yesno`) wraps real `readline`/stdin, also with no
  injection seam. Noticed in passing (not fixed here, out of scope for a docs
  pass): the `switch` on the answer has no `break`/`return` per case, so
  execution always falls through past the first matching case toward
  `default`, calling `resolve()` a second time — harmless today only because
  Promise semantics make the first `resolve()` win, but it's real latent bug
  shape, not just an untested branch.
- **MSAL/DevOps/Power Platform SDK calls** are unit-tested only via mocks —
  there's no integration tier against a real Azure AD tenant, DevOps org, or
  Dataverse environment. Building one would need real test-tenant credentials
  in CI, a genuine security/cost tradeoff, not a quick add.
- **Everything outside `coe-cli`** — the actual CoE Starter Kit content: Power
  Automate flows, canvas/model-driven apps, Power BI reports across every
  `CenterofExcellence*`/`ALMAccelerator*` solution folder — has no automated
  test framework, and there's no obvious lightweight one; it would need a live
  Dataverse environment plus a flow-execution harness. This is the single
  biggest coverage gap in the repository, and also the most expensive to close.

## Standard going forward

1. New or changed command logic gets a matching `test/.../*.spec.ts`, using
   whichever existing pattern fits (pure-function table test vs. mocked-SDK
   test).
2. New I/O-touching code goes through the constructor-injection convention so
   it *stays* testable — don't add a class that calls `child_process`/`fs`/a
   SDK client directly without an overridable seam.
3. `npm run build`, `npm run lint`, and `npm test` must all pass before merge
   — now enforced in CI (see `TODO-Craig-Detailed.md` §0 in the repo root).
4. No hard coverage-% gate proposed yet. The codebase is pre-existing and
   unevenly covered; a blanket gate would either block unrelated work or need
   scoping to changed files only. Revisit once coverage is trending in a
   known direction rather than gating on today's uneven baseline.
