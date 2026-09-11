---
title: Copilot Context Cleanup
description: Applied cleanup record for Copilot customization files in the LCE Menu Admin API repository
author: GitHub Copilot
ms.date: 2026-08-05
ms.topic: reference
keywords:
  - github copilot
  - cleanup
  - customization
estimated_reading_time: 8
---

## Scope

* Workspace root: `c:/source/lce-menu-admin-api`
* Source audit path: `docs/ai/copilot-context-audit.md`
* Cleanup boundaries:
  * `.github/copilot-instructions.md`
  * `.github/copilot/**/*.md`
  * `.github/skills/**`
  * `.github/prompts/**/*.prompt.md`
  * `.github/README.md`
  * `docs/**/*.md`
* Repository-state limitations:
  * preexisting deleted `.github/agents/*.agent.md` files were preserved as-is
  * `.github/instructions/*.instructions.md` were not modified by hard user requirement

## Preexisting Changes

* Deleted before this cleanup slice:
  * `.github/agents/PostgresExpert.agent.md`
  * `.github/agents/acceptance-tests.agent.md`
  * `.github/agents/aspire-expert-agent.md`
  * `.github/agents/unit-tests.agent.md`
* Untracked alongside cleanup:
  * `.copilot-tracking/research/subagents/2026-08-05/`
  * `.github/skills/copilot-context-auditor/`
  * `docs/ai/copilot-context-audit.md`

## Planned Operations

1. `modify` `.github/prompts/dbdiagram.prompt.md` for `AUDIT-001`
2. `modify` `.github/copilot-instructions.md` for `AUDIT-002`, `AUDIT-003`, `AUDIT-004`
3. `modify` `.github/README.md`, `.github/skills/README.md`, and `.github/prompts/plan-controller-parity-implementation.prompt.md` for `AUDIT-005`
4. `modify` coverage and mutation prompts or skills for `AUDIT-006` and `AUDIT-009`
5. `delete` `.github/copilot-coding-standards.md` for `AUDIT-004`
6. `modify` `.github/skills/copilot-context-auditor/SKILL.md` for `AUDIT-008`
7. `modify` `docs/ai/copilot-context-audit.md` and `create` `docs/ai/copilot-context-cleanup.md`

## Applied Changes

### CLEANUP-001

* Audit finding IDs: `AUDIT-001`
* Operation: `modify`
* Source: `.github/prompts/dbdiagram.prompt.md`
* Destination: same file
* Evidence: the prompt mixed `docs/diagrams/menu-admin-db-diagram.*` with nonexistent `docs/menu-admin-db-diagram.*` paths
* Exact change: rewrote verification and deliverable references to use only `docs/diagrams/menu-admin-db-diagram.md` and `docs/diagrams/menu-admin-db-diagram.mmd`
* Content preserved: canonical command, task-ordering, and generator requirements
* Why the change reduces unnecessary context: removes contradictory path instructions and avoids duplicate path families
* Activation after cleanup: manual prompt invocation
* Validation: grep confirmed only `docs/diagrams/...` remains; diagnostics clean
* Status: `Applied`

### CLEANUP-002

* Audit finding IDs: `AUDIT-002`, `AUDIT-003`, `AUDIT-004`
* Operation: `modify`
* Source: `.github/copilot-instructions.md`
* Destination: same file
* Evidence: root file carried broad `docs/ai/**/*.md` guidance and duplicated low-level repository, OpenAPI, and DTO rules already owned elsewhere
* Exact change: added markdown frontmatter, narrowed broad docs guidance to named entry documents, removed duplicated low-level rules, and moved the remaining `.editorconfig` rule into the root file
* Content preserved: routing, architecture boundaries, audit logging, unique exception-message rule, planning workflow, auth-constant centralization, and PR markdown output rule
* Why the change reduces unnecessary context: keeps always-on context to routing and universal constraints only
* Activation after cleanup: repository-wide root instruction loading
* Validation: grep confirmed removed wildcard and duplicated rule text no longer appears in the root file; diagnostics clean
* Status: `Applied`

### CLEANUP-003

* Audit finding IDs: `AUDIT-005`
* Operation: `modify`
* Source: `.github/README.md`, `.github/skills/README.md`, `.github/prompts/plan-controller-parity-implementation.prompt.md`
* Destination: same files
* Evidence: stale OpenSpec claims, deprecated skills listed without deprecation labels, and a dead `docs/parity/order` example path
* Exact change: rewrote `.github/README.md` around current assets only, marked deprecated skills in the skills index, and replaced the dead parity example with supplied-artifacts wording
* Content preserved: high-level directory purpose and current skill index intent
* Why the change reduces unnecessary context: removes dead discovery paths and outdated workflow framing
* Activation after cleanup: documentation-only for READMEs, manual prompt invocation for the parity-planning prompt
* Validation: grep found no remaining `OpenSpec`, `openspec`, or `docs/parity/order` matches under `.github`
* Status: `Applied`

### CLEANUP-004

* Audit finding IDs: `AUDIT-006`, `AUDIT-009`
* Operation: `modify`
* Source:
  * `.github/skills/menu-admin-adhoc-quality/SKILL.md`
  * `.github/skills/menu-admin-api-filtered-coverage/SKILL.md`
  * `.github/skills/menu-admin-api-mutation-fast/SKILL.md`
  * `.github/skills/menu-admin-cobertura-source-only-analysis/SKILL.md`
  * `.github/skills/menu-admin-coverage-hardening-autonomous/SKILL.md`
  * `.github/skills/menu-admin-coverage-mutation-check/SKILL.md`
  * `.github/prompts/fix-tests.prompt.md`
  * `.github/prompts/fix-mutations.prompt.md`
  * `.github/prompts/test-coverage-hardening.prompt.md`
* Destination: same files
* Evidence: repeated `/caveman full`, `Auto-Clarity`, and strict `KEY=VALUE` boilerplate plus malformed trailing fences in deprecated skill files
* Exact change: replaced repeated boilerplate with pointers to `.github/skills/coverage-output-contract.md` and removed stray fence closers from deprecated skill files
* Content preserved: unique commands, key lists, thresholds, and final-report exceptions
* Why the change reduces unnecessary context: centralizes the default output contract in one file and keeps local files focused on unique workflow content
* Activation after cleanup: task-scoped skill or manual prompt activation
* Validation: grep found no remaining local copies of the duplicated output-contract strings in the touched files; diagnostics clean
* Status: `Applied`

### CLEANUP-005

* Audit finding IDs: `AUDIT-004`
* Operation: `modify`
* Source: `.github/copilot/testing-and-coverage.md`, `.github/copilot/dotnet-csharp-best-practices.md`
* Destination: same files
* Evidence: test-specific guidance still leaked into the general C# file after the old coding-standards file was removed
* Exact change: moved the remaining Moq and scenario-validation test guidance into the testing owner and removed test-specific bullets from the general C# best-practices file
* Content preserved: test behavior rules remained intact under the testing owner; language-level C# guidance remained intact in the general file
* Why the change reduces unnecessary context: gives test conventions one clear routed owner
* Activation after cleanup: routed section-file loading based on task domain
* Validation: grep confirmed the cited test-guidance strings remain in `testing-and-coverage.md` and no longer appear in `dotnet-csharp-best-practices.md`
* Status: `Applied`

### CLEANUP-006

* Audit finding IDs: `AUDIT-004`
* Operation: `delete`
* Source: `.github/copilot-coding-standards.md`
* Destination: deleted
* Evidence: its remaining unique formatting rule moved to `.github/copilot-instructions.md`, and its test-specific guidance moved or remained under `.github/copilot/testing-and-coverage.md`
* Exact change: removed the redundant file after clearing its last live reference in `.github/README.md`
* Content preserved: repository-wide `.editorconfig` guidance in the root instruction and test guidance in the testing owner
* Why the change reduces unnecessary context: removes a stale duplicate file and its extra discovery surface
* Activation after cleanup: not applicable
* Validation: grep found no remaining references to `.github/copilot-coding-standards.md`
* Status: `Applied`

### CLEANUP-007

* Audit finding IDs: `AUDIT-008`
* Operation: `modify`
* Source: `.github/skills/copilot-context-auditor/SKILL.md`
* Destination: same file
* Evidence: the skill previously included a blanket `docs/**/*.md` scan in its main in-scope file list
* Exact change: replaced that blanket scope with a direct-reference or audit-claim validation rule for additional docs and scripts
* Content preserved: cleanup boundaries still allow `docs/**/*.md` modifications when cleanup mode changes a documentation file
* Why the change reduces unnecessary context: narrows audit discovery to evidence-backed docs and scripts
* Activation after cleanup: skill activation only
* Validation: grep confirmed the old blanket-scan item is gone from the main in-scope list; diagnostics clean
* Status: `Applied`

### CLEANUP-008

* Audit finding IDs: `AUDIT-001` through `AUDIT-009`
* Operation: `modify` and `create`
* Source: `docs/ai/copilot-context-audit.md`
* Destination: `docs/ai/copilot-context-audit.md`, `docs/ai/copilot-context-cleanup.md`
* Evidence: the earlier audit report was in free-form narrative and cleanup mode requires structured post-cleanup reporting
* Exact change: rewrote the audit into the required structured format and created this cleanup report
* Content preserved: original finding substance, evidence themes, and repo-state caveats
* Why the change reduces unnecessary context: converts ad hoc research into stable, reviewable cleanup artifacts
* Activation after cleanup: documentation only
* Validation: final frontmatter, local-link, `applyTo`, stale-reference, and diagnostics checks completed cleanly
* Status: `Applied`

## Files Created

* `docs/ai/copilot-context-cleanup.md` via `CLEANUP-008`

## Files Modified

* `.github/prompts/dbdiagram.prompt.md` via `CLEANUP-001`
* `.github/copilot-instructions.md` via `CLEANUP-002`
* `.github/README.md` via `CLEANUP-003`
* `.github/skills/README.md` via `CLEANUP-003`
* `.github/prompts/plan-controller-parity-implementation.prompt.md` via `CLEANUP-003`
* `.github/skills/menu-admin-adhoc-quality/SKILL.md` via `CLEANUP-004`
* `.github/skills/menu-admin-api-filtered-coverage/SKILL.md` via `CLEANUP-004`
* `.github/skills/menu-admin-api-mutation-fast/SKILL.md` via `CLEANUP-004`
* `.github/skills/menu-admin-cobertura-source-only-analysis/SKILL.md` via `CLEANUP-004`
* `.github/skills/menu-admin-coverage-hardening-autonomous/SKILL.md` via `CLEANUP-004`
* `.github/skills/menu-admin-coverage-mutation-check/SKILL.md` via `CLEANUP-004`
* `.github/prompts/fix-tests.prompt.md` via `CLEANUP-004`
* `.github/prompts/fix-mutations.prompt.md` via `CLEANUP-004`
* `.github/prompts/test-coverage-hardening.prompt.md` via `CLEANUP-004`
* `.github/copilot/testing-and-coverage.md` via `CLEANUP-005`
* `.github/copilot/dotnet-csharp-best-practices.md` via `CLEANUP-005`
* `.github/skills/copilot-context-auditor/SKILL.md` via `CLEANUP-007`
* `docs/ai/copilot-context-audit.md` via `CLEANUP-008`

## Files Moved

* None

## Files Deleted

* `.github/copilot-coding-standards.md` via `CLEANUP-006`

## Findings Resolved

* `AUDIT-001`
* `AUDIT-003`
* `AUDIT-004`
* `AUDIT-005`
* `AUDIT-006`
* `AUDIT-008`
* `AUDIT-009`

## Findings Skipped

* `AUDIT-007`
  * Exact reason: the targeted custom-agent file was not present in the current working tree because `.github/agents/*.agent.md` files were already deleted before this cleanup slice.
  * Failed prerequisite: live source file to revalidate and update
  * Manual review required: only if those agent files are restored later

## Validation Results

* Frontmatter delimiter and required-field checks across edited files: passed
* Local markdown-link resolution across edited files: passed
* `.github/instructions/*.instructions.md` `applyTo` presence checks: passed
* Stale-reference search outside audit docs for `.github/copilot-coding-standards.md`, `docs/menu-admin-db-diagram`, and `docs/parity/order`: passed
* VS Code diagnostics on the audit and cleanup reports: passed

## Remaining Risks

* `AUDIT-002` remains partially open because the protected always-on `.instructions.md` files still apply repository-wide.
* `/skills` menu visibility remains unverified through the current VS Code Copilot Chat session.

## Final Repository Structure

```text
.github/
├── copilot-instructions.md
├── instructions/
├── copilot/
├── prompts/
├── skills/
│   ├── coverage-output-contract.md
│   └── <skill>/SKILL.md
└── README.md

docs/ai/
├── copilot-context-audit.md
└── copilot-context-cleanup.md
```