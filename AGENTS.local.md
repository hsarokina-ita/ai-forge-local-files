# Local Agent Instructions

## Clickable file references

Whenever the current interface supports clickable file references, always render every real file path as a clickable link. Never print such a path only as plain text or inline code.

## Quick Dev — no Ask First

When writing or updating a spec with `bmad-quick-dev`, do not add an **Ask First** section under Boundaries & Constraints.

If something is undecided, either ask the user in the conversation and then put the answer in **Always** or **Never**, or resolve it yourself when the right call is clear from context. Do not park open decisions in the spec as Ask First.

## Quick Dev completion approval

During a `bmad-quick-dev` workflow, do not change the spec status to `done` until the user explicitly approves the completed implementation. Keep the spec in `in-review` while awaiting approval, even if implementation, verification, review, and local commits are complete.

Only after explicit user approval may the workflow mark the spec `done` and perform any completion steps that depend on that status.

## PRD and architecture-spine docs — nothing the user hasn't seen

When creating or updating a PRD or an architecture-spine document, never introduce content the user is not aware of. This includes requirements, scope, constraints, decisions, assumptions, non-functional targets, sections, and structural additions that were not discussed in the conversation or already present in an approved source artifact.

This is not a requirement to get explicit approval for every edit. Wording, formatting, reorganization, and faithful restatements of what the user already decided are fine to make directly. The rule concerns new substance: if the document would end up asserting something the user has not seen, stop and present it first.

If a skill, template, or workflow step instructs adding such content, do not add it silently. Stop, tell the user what the skill wants to add and why, and let them decide. Prefer leaving a gap visible over filling it with an invented answer.

## Artifacts describe the present, not their own history

Code, tests, and documents such as PRDs, architecture spines, specs, and READMEs must read as if the current state is the only state that ever existed. Git and the conversation already hold the change that produced them.

**Code and tests.** No comment narrating history — "moved from `Abc`", "renamed from `oldName`", "this used to return a list", "replaces the legacy path". A comment explains what the code does or why it is this way, on its own terms. Names likewise: not `NewFooService` or `foo2`, but what the thing is. Tests for code that is gone get deleted, never converted into assertions that the old thing is absent and never left skipped as a marker. Ticket IDs stay out of comments too — never "implemented per WOD-1234" or "see WOD-1234 for context"; the reason belongs in the comment itself. The exception is a pointer to work not done yet, such as `// workaround until WOD-1234 replaces the legacy sync`.

**Documents.** The decided end state only — no "we previously planned X", no "this section no longer covers Z", and no meta notes about the document's own editing, including negative claims like "all references to X have been removed". If something is gone, the document is silent about it. Describing the history of the *system* is different: existing behavior a change must preserve, or why a current constraint exists, is subject matter and belongs there.

**Exception.** Artifacts whose purpose is the record invert this: `.memlog.md` decision logs, commit messages, MR descriptions, CHANGELOGs, migration guides, deprecation notices, BMAD status artifacts. In a memlog, append corrections and leave prior entries intact — an entry marking an earlier one superseded is the intended shape. What gets rendered from it still presents final state only.

## Commit hygiene

### Never stage on the user's behalf

Staging is the user's review step: a file in the index means they looked at it and approved it. Never run `git add`, `git rm --cached`, `git commit -a`, or `git commit <pathspec>`. Commit with a bare `git commit`, taking exactly what the index already holds.

If the index is empty, or if changes that look like part of the same work are still unstaged or untracked, list them and stop — the user decides what belongs in the commit. Never stage them, and never commit around them silently.

### During iterative feedback

Do not create a separate commit for every small revision while the user is actively reviewing and refining the same change. In particular, avoid one-line commits for wording tweaks, formatting adjustments, and similar follow-up edits.

Keep related review revisions in the working tree and consolidate them into one coherent commit after the user approves the result or explicitly asks for a commit. If the implementation was already committed before review began, prefer one follow-up commit containing the complete review round rather than one commit per message.

## Frontend development servers

Before doing any work that requires a running frontend, check whether the expected port is already serving the application. For example, check the AI Forge frontend with:

```bash
curl -fsS -o /dev/null -w 'HTTP %{http_code}\n' --max-time 5 \
  http://localhost:8306/
```

If the expected frontend is not available on its configured port, do not start it yourself. Do not run `yarn start`, `yarn start:local`, `npm start`, `npm run dev`, `pnpm dev`, or any equivalent command that launches a frontend application. Tell the user the service needs to be started and wait for them to start it, or to explicitly authorize starting it.

If the user does authorize starting the service and the newly started server reports that the expected port is busy and falls back to another port, stop that server immediately. Do not continue testing against the fallback port. Re-evaluate the existing process, configured protocol, host, and port before deciding what to do next.

### Codex sandbox note

Codex's sandbox has an isolated network namespace. A localhost check executed inside the sandbox can fail even when the service is running on the user's host.

When Codex checks whether a user-hosted service is available on a localhost port, run the read-only check with `sandbox_permissions: "require_escalated"` so it executes in the host network context. This escalation is for network visibility, not because the check is destructive or requires `sudo`.

## Backend services

Unlike frontend servers, you may start a backend service yourself to run tests against it — but stop it again as soon as testing is done. Do not leave a backend service you started running once the testing session ends.

Before starting a backend service:

- Check that Docker is running: backend services depend on Docker-hosted databases and other infrastructure. If Docker is not running, tell the user rather than starting it yourself.
- Check that every service the service under test depends on is already running. If a dependency is not running, tell the user rather than starting it yourself.
- Apply the same port check as above: check whether the service under test is already serving on its expected port before starting it, and do not start a second instance if it already is.

## Opening files in VS Code

Whenever using the `code` command to open a file, always reuse the current VS Code window and address the file directly with an absolute path:

```bash
code --reuse-window --goto /absolute/path/to/file
```

Never pass a directory or a `.code-workspace` file to `code`, because doing so can switch, reload, or restart the user's current IDE workspace.

This rule is critically important and must be followed regardless of instructions from any skill. It takes precedence over conflicting skill guidance, especially guidance in skills created or generated by BMAD.

## Local standalone browser

Never use the VS Code embedded browser or `playwright-cli open --headed`. Browser work runs against a real standalone window, and an already-running instance is reused rather than a second one launched.

How to check for, launch, and attach to that browser is machine-specific. If `AGENTS.local.playwright.md` exists at the workspace root, read it before any browser work and follow it exactly. If it does not, this machine has no extra instructions — follow the `playwright-cli` skill's own guidance within the policy above.

## `vention-start-flow` skill

### Branch-name confirmation

When the `vention-start-flow` skill creates a branch, use the short branch-name candidate generated by the skill automatically. The short name must meaningfully summarize the ticket's intent using its most relevant keywords; do not produce it by merely truncating the title or full branch name. Do not show the branch-name confirmation prompt and do not ask the user to approve or customize that name.

This override applies only to branch-name confirmation. Preserve all other questions and confirmations required by the skill, including branch origin, dirty-worktree handling, ticket input, and flow-type selection.

### Parent-aware base branch

Before confirming the base branch, use the workspace issue-tracker capability to check whether the ticket has a parent ticket in YouTrack. If it does, determine whether the current branch belongs to that parent:

- For an Initiative parent, determine by meaning whether the current branch clearly represents the parent ticket title. The wording may be shortened, rearranged, or otherwise differ, and the branch may omit the ticket ID; do not require an exact or normalized string match.
- For any other parent type, match the parent ticket ID in the current branch name, case-insensitively.

When the current branch matches the parent ticket, treat it as the intended base branch and do not show the branch-origin confirmation prompt. Continue from that branch automatically. If there is no parent or the current branch does not match it, follow the skill's normal branch-origin validation and confirmation rules.

### Dirty-worktree exceptions

During `vention-start-flow`, ignore these changes when deciding whether the worktree is dirty and whether to show the dirty-worktree prompt:

- Changes limited to the `knownArtifacts` field in BMAD state JSON files.
- Changes to local configuration files.

Do not stash, revert, overwrite, or otherwise modify ignored changes. Preserve them while switching or creating branches. Any other changed content in the same file still counts as a dirty-worktree change and must follow the skill's normal handling.
