# argos — autonomous repository maintainer

**argos** is an AI maintainer that watches the repositories listed in
[`REPOSITORIES.md`](./REPOSITORIES.md) and works on them like a (tireless)
team member: it triages issues, proposes implementation plans, writes the
code, runs the tests, opens pull requests, answers review comments, fixes
CI, and can verify a web app's UI end-to-end in a real browser.

It acts on GitHub as **@argos-code** (you may also see `argos-code[bot]`
— same actor, different credential).

---

## Managing which repositories argos watches

This repository is the control plane. [`REPOSITORIES.md`](./REPOSITORIES.md)
holds a table of managed repositories:

- **Add a repository**: open a pull request adding a row. Once merged,
  argos bootstraps the repo (clones it, learns how to build and test it,
  generates its per-repo configuration) and starts watching it.
- **Enable UI testing**: set the *UI testing* column to `yes`. Argos will
  learn how to boot the app, propose a flows file (see below), and run
  browser tests on demand.
- **Remove a repository**: delete its row. Argos stops watching it but
  keeps its stored history; permanent deletion is an operator-only CLI
  action.

Changes made from the operator CLI are mirrored back into the file
automatically, so the table always reflects the live state. Edit only the
table between the markers; the surrounding prose is yours.

---

## How to work with argos on a watched repository

Talk to it in plain language — mention **@argos-code** in an issue or PR
comment, assign it an issue, or apply the repo's activation label. A
comment router (with memory of the whole repo conversation) figures out
what you mean; you don't need exact command syntax.

### The issue lifecycle

1. **Triage** — ask argos to look at an issue (*"@argos-code take a look
   at this"*). It analyzes the issue against the codebase and either asks
   blocking questions, proposes a plan, or classifies it as an easy fix.
2. **Plan approval (human gate)** — for normal work argos posts an
   implementation plan with stable IDs (AC1, P1.T2, R3…). Nothing is
   implemented until a maintainer approves (*"go ahead"*, *"lgtm"*) or
   requests changes (*"drop AC2, split P1.T3"* → it replans).
3. **Implementation** — argos codes in an isolated container, validates
   (format/lint/typecheck/tests/build), repairs failures, and opens a PR.
4. **After the PR** — argos watches CI and fixes red builds, answers
   review comments (implementing accepted suggestions), and keeps going
   until merge.

### Things you can ask on a pull request

| Ask | What happens |
|---|---|
| *"review this PR"* | Fresh-context code review with findings (works on any PR, not just argos's) |
| *"revise: …"* | Argos pushes the requested changes to the PR branch |
| *"put your fixes in a separate PR on top"* | Opens a stacked suggestion PR instead of touching your branch |
| *"squash your commits"* | Mechanically collapses its commits into one (content-verified) |
| *"can you fix CI?"* | Diagnoses and fixes the failing checks |
| *"test the ui"* | Runs the repo's UI flows against the PR in a real browser (see below) |
| *"what's the status?"* | Posts a status summary |
| *"keep going"* / *"try again"* | Resumes an issue argos parked as BLOCKED |
| *"cancel"* | Stops and forgets the issue |

State-changing requests are permission-gated: only repository
collaborators can approve plans, trigger runs, squash, cancel or resume.

### How argos communicates

Argos is deliberately quiet. It reacts 👀 to show it noticed you, 👍 on a
review comment it implemented, and only writes a comment when it carries
real information (a plan, a question, a result, a disagreement). If it
hits something it can't resolve — a protected file, a failing loop — it
parks the issue as BLOCKED and tells you why; a maintainer's *"resume"*
is treated as approval to proceed.

Certain paths (CI workflows, migrations, lockfiles, secrets) are
**protected**: argos won't push changes touching them unless a maintainer
explicitly approves.

---

## UI flow testing

For repositories with *UI testing* enabled, argos maintains browser-level
end-to-end tests driven by a **human-owned contract file** committed to
the repo: `.argos/ui-flows.md`.

- The file describes, in plain markdown, how to boot the app (its `Setup`
  section) and which user flows must keep working — written as **required
  behavior**, not screen details, so cosmetic changes don't break tests.
- When UI testing is first enabled, argos explores the running app in a
  real browser, writes an initial flows file, proves it green, and opens
  a **seed PR**. Merging that PR activates the feature — the file is
  yours from then on.
- Any collaborator can comment *"@argos-code test the ui"* on any PR.
  Argos checks out the PR, runs every flow headless (Chromium), and posts
  a single pass/fail report with error details.
- Editing the flows file in a PR is the intended workflow: argos
  recompiles the browser tests automatically the next time it runs. The
  compiled tests, browsers, and artifacts all live on argos's side — the
  repo only ever contains the markdown contract.
- Runs are deterministic and mutate nothing: flows may fill forms and
  open dialogs, but never publish, save, or invite.

---

## Under the hood (short version)

Argos is a self-hosted daemon. Every phase runs the Claude Code CLI in a
locked-down container (resource caps, capability drops, no host access);
every guarantee that matters — protected paths, commit-before-push,
plan approval — is enforced deterministically by the orchestrator, not
just by prompts. Humans stay in the loop at the moments that matter.
