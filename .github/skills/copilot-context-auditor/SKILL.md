---
name: copilot-context-auditor
description: >
  Audit GitHub Copilot customization files and their referenced documentation
  for duplicated instructions, conflicting rules, excessive always-on context,
  weak path scoping, stale references, and avoidable token or AI credit usage.
  Use only when the user explicitly asks to audit, optimize, consolidate,
  validate, clean up, or reduce Copilot instructions, skills, prompts, agents,
  AGENTS.md files, documentation, or Copilot context usage. Do not use for
  ordinary coding, documentation, code review, planning, implementation,
  deployment, or troubleshooting tasks.
---

# Copilot Context Auditor

## Purpose

Minimize GitHub Copilot context and AI credit usage without removing required
repository knowledge, safety rules, architecture constraints, development
standards, or validation requirements.

This skill runs inside VS Code Copilot Chat against the currently open
workspace.

Do not optimize by guessing. Every finding and every cleanup change must be
supported by evidence from the current workspace.

## Operating Modes

This skill has two modes:

1. Audit mode
2. Cleanup mode

Audit mode is the default.

Cleanup mode is enabled only when the user explicitly asks to apply, implement,
perform, or execute cleanup based on an existing audit.

Do not infer cleanup permission from a request to audit, review, inspect,
analyze, validate, or recommend improvements.

## Workspace Boundary

Operate only within the currently open VS Code workspace.

Do not inspect parent directories, sibling repositories, user-profile
customizations, organization-level instructions, or unrelated workspaces unless
the user explicitly includes them and they are accessible through the current
workspace.

Do not claim to have inspected customization sources that are not accessible
through the current VS Code Copilot Chat session.

## In-Scope Files

Inspect all existing files matching these locations when present:

- `.github/copilot-instructions.md`
- `.github/instructions/**/*.instructions.md`
- `.github/copilot/**/*.md`
- `.github/skills/**/SKILL.md`
- `.github/prompts/**/*.prompt.md`
- `.github/agents/**/*.agent.md`
- `.github/chatmodes/**/*.chatmode.md`
- `.github/hooks/**/*`
- `.vscode/mcp.json`
- `.vscode/settings.json`
- `.vscode/extensions.json`
- `AGENTS.md`
- `**/AGENTS.md`
- `CLAUDE.md`
- `**/CLAUDE.md`
- `README.md`
- `**/README.md`
- Markdown files referenced by an instruction, skill, prompt, agent, AGENTS.md,
  CLAUDE.md, or other in-scope customization file
- scripts referenced by an instruction, skill, prompt, agent, AGENTS.md,
  CLAUDE.md, or supporting document
- configuration files that explicitly register or configure Copilot skills,
  instructions, agents, prompts, tools, or MCP servers

Inspect additional `docs/**/*.md` or `scripts/**/*` files only when a
customization file references them directly or a specific audit claim requires
them for validation.

Do not assume an ordinary Markdown file enters Copilot context merely because it
exists.

## Out-of-Scope Files

Do not inspect generated output, dependencies, binaries, coverage reports,
package caches, build artifacts, source code, tests, infrastructure, pipelines,
or database assets unless inspection is required to validate a specific claim
made by an in-scope customization file.

Do not change out-of-scope files in either operating mode.

## Hard Rules

1. Do not invent token limits, token counts, AI credit costs, savings estimates,
   file-loading behavior, or Copilot behavior.
2. Do not claim a file is loaded automatically unless its activation mechanism
   provides evidence for that conclusion.
3. Do not treat every Markdown file as Copilot context.
4. Do not delete, move, rename, shorten, or rewrite files during audit mode.
5. Do not remove security, privacy, production-safety, compliance,
   destructive-operation, architecture-boundary, database-safety, testing,
   coverage, or validation rules merely to reduce context.
6. Do not weaken `MUST`, `MUST NOT`, `NEVER`, `REQUIRED`, or equivalent rules.
7. Do not replace precise repository rules with generic best practices.
8. Do not treat file length alone as proof of waste.
9. Do not recommend moving text unless the destination and activation mechanism
   are identified.
10. Do not duplicate detailed guidance into repository-wide instructions.
11. Do not perform broad source-code exploration when customization files
    provide sufficient evidence.
12. Prefer deterministic scripts or terminal commands for mechanical validation.
13. Preserve existing behavior unless a verified conflict requires correction.
14. Separate verified findings from recommendations.
15. If evidence is unavailable, use `NOT-VERIFIED`.
16. Never report approximate savings as factual token or credit measurements.
17. Never store secrets, credentials, tokens, private customer data, production
    connection strings, or sensitive personal data in generated reports.
18. Do not assume file precedence when precedence is not documented or proven.
19. Do not assume a referenced document is automatically loaded merely because
    an instruction links to it.
20. Do not claim that a customization was active in the current chat unless
    VS Code exposes evidence confirming it.
21. Do not use GitHub Copilot CLI commands as a required part of this workflow.
22. Do not modify application behavior while optimizing Copilot customization.
23. Do not create compatibility copies of the same detailed instructions in
    multiple locations.
24. Do not use subjective language such as bloated, useless, excessive, obvious,
    or unnecessary without quoted evidence and a defined reason.
25. Do not ask for confirmation between safe cleanup steps after the user has
    explicitly invoked cleanup mode.

## Evidence Standard

Every finding must include:

- exact source path
- exact quoted text
- applicable line numbers when available
- activation or reference evidence
- explanation tied directly to the evidence
- exact proposed change
- validation method

For duplicate or conflicting guidance, quote every relevant location.

Do not summarize a rule as evidence when the exact text can be quoted.

If a conclusion depends on behavior that cannot be verified from the workspace,
state:

`Not verified through the current VS Code Copilot Chat session.`

## Repository Tool Strategy

Use VS Code workspace search and file-read tools before terminal commands.

Use terminal commands only when they provide deterministic results more
efficiently, including:

- listing matching files
- measuring line and byte counts
- locating Markdown links
- locating file references
- checking whether referenced paths exist
- detecting exact duplicate text
- validating YAML frontmatter
- validating `applyTo` patterns syntactically
- checking Git status or Git diff
- generating audit and cleanup reports

Use read-only commands during audit mode.

Do not install dependencies, extensions, packages, tools, or global utilities.

Do not run commands that modify application code, dependencies, infrastructure,
pipelines, databases, or production configuration.

## Audit Mode

Audit mode is read-only except for creating or updating the audit report.

The only file audit mode may create or update is:

`docs/ai/copilot-context-audit.md`

Do not modify any existing customization file during audit mode.

### Audit Process

#### 1. Establish Repository State

Record:

- workspace root
- current Git branch, when available
- whether the working tree already contains changes
- whether an earlier audit exists
- any scope limitations encountered

Do not modify, discard, stage, or commit existing changes.

#### 2. Inventory Customizations

Find every in-scope file.

For each file, record:

- exact path
- customization type
- activation mechanism
- declared task scope
- declared path scope
- files referenced
- files referencing it
- line count
- byte count
- whether it appears to be:
  - repository-wide
  - path-scoped
  - task-scoped
  - manually invoked
  - skill-triggered
  - agent-selected
  - auto-discovered
  - supporting documentation only
  - not verified

Do not interpret ordinary Markdown documentation as automatically loaded unless
another customization mechanism provides evidence that it enters context.

#### 3. Build the Context Graph

Build a directed context graph showing:

- always-on instructions
- path-scoped instructions
- task-scoped instructions
- manually invoked prompt files
- relevant skill activation
- explicitly selected agents
- AGENTS.md hierarchy
- CLAUDE.md compatibility files
- linked supporting documentation
- scripts invoked by prompts, skills, agents, or instructions
- MCP or tool configuration referenced by agents
- files that can cause another file to be inspected

For every edge, identify the source evidence.

Distinguish:

- automatically included
- conditionally applicable
- selected by the user
- selected by relevance
- explicitly referenced
- supporting documentation only
- not verified

#### 4. Detect Duplicate Guidance

Compare semantic rules across in-scope files.

For every duplicate, provide:

- source path and exact quoted text
- duplicate path and exact quoted text
- whether the relationship is:
  - identical
  - overlapping
  - conflicting
- which file should own the detailed rule
- which file, if any, should retain a concise pointer
- why the proposed ownership matches the activation scope

Do not classify examples, validation commands, or repository-specific
implementation details as duplicates of a high-level rule when they add
necessary information.

#### 5. Detect Over-Broad Context

Flag evidence-supported instances of:

- repository-wide rules relevant only to one language
- repository-wide rules relevant only to one project or folder
- repository-wide rules relevant only to tests
- repository-wide rules relevant only to deployment or infrastructure
- instructions requiring many documents to be read before every task
- broad `applyTo` patterns where narrower patterns are supported by evidence
- README or architecture content copied into always-on instructions
- agents with tools unrelated to their declared role
- skill descriptions broad enough to trigger for unrelated requests
- prompts requiring full-workspace discovery before targeted inspection
- instructions requiring narration or repeated summaries
- rules requiring information already provided by the task to be restated
- large examples that can live in referenced documentation
- full logs or chat transcripts embedded in reusable Markdown
- detailed troubleshooting content placed in always-on instructions
- repeated output-format requirements copied across multiple assets
- permanent context containing one-time task requirements

For every finding, quote the triggering text and identify the activation scope
that makes it broader than necessary.

#### 6. Detect Missing or Weak Scoping

Check whether:

- `.instructions.md` files have valid YAML frontmatter
- applicable `.instructions.md` files declare `applyTo`
- `applyTo` patterns match the documented purpose
- skill descriptions define when the skill should activate
- skill descriptions avoid activation during unrelated work
- prompts are manually invoked rather than treated as permanent instructions
- agents have clear roles, boundaries, and tool scope
- detailed standards are linked or conditionally loaded rather than copied into
  global instructions
- nested `AGENTS.md` files are located at an appropriate directory scope
- referenced paths exist
- Markdown links resolve relative to the containing file
- file names and locations follow the repository's existing convention
- customization files identify expected inputs and outputs
- customization files identify validation requirements
- cleanup or mutation workflows require explicit user intent

Do not propose a new convention if the repository already has a consistent,
working convention.

#### 7. Detect Conflicts and Ambiguity

Find rules that:

- directly contradict one another
- define inconsistent precedence
- use inconsistent terminology for the same concept
- specify different commands for the same validation
- disagree about architecture boundaries
- disagree about test or coverage execution
- disagree about deployment or database behavior
- mix optional language with mandatory language
- reference obsolete paths
- reference unavailable tools
- reference obsolete project names
- reference obsolete versions
- contain unresolved placeholders
- contain conflicting output requirements
- contain conflicting modification permissions

Every conflict must quote all conflicting locations.

Do not choose a winner without repository evidence. If precedence cannot be
verified, classify the finding as `NOT-VERIFIED` and recommend manual review.

#### 8. Detect Low-Value Permanent Context

Flag evidence-supported content that is:

- generic model behavior
- motivational filler
- repeated purpose statements
- historical conversation copied into durable guidance
- stale troubleshooting narration rather than a verified resolution
- redundant output-format rules
- generic coding advice with no repository-specific constraint
- task-specific detail placed in permanent global instructions
- duplicated explanation that adds no constraint, example, exception, or
  validation instruction

Do not flag repository-specific facts merely because they are verbose.

Do not flag a safety rule merely because it appears in more than one
safety-sensitive workflow. Instead, determine whether central ownership plus a
reference would preserve enforcement.

#### 9. Inspect Skills

For each skill, verify:

- folder contains `SKILL.md`
- YAML frontmatter exists
- `name` is present
- `description` is present
- skill name is clear and appropriately scoped
- description identifies triggering requests
- description avoids unrelated activation
- workflow is concrete
- inputs are defined
- outputs are defined
- modification boundaries are defined
- validation is defined
- supporting scripts or resources exist
- referenced files exist
- the skill does not duplicate always-on instructions
- deterministic work is delegated to scripts where appropriate
- mutation requires explicit user intent

Do not run another skill merely because it is found during this audit.

#### 10. Inspect Prompt Files

For each prompt file, verify:

- intended task is clear
- invocation is manual or otherwise explicitly defined
- required inputs are identified
- output contract is clear
- repository discovery is targeted
- it does not duplicate permanent instructions
- it does not contain stale file paths
- it does not require unnecessary narration
- it does not request unsupported token or credit calculations
- mutation boundaries are clear
- validation requirements are explicit where needed

#### 11. Inspect Custom Agents

For each custom agent, verify:

- YAML frontmatter exists
- name and description are present
- role is narrow and clear
- tool access matches the role
- read boundaries are clear
- modification boundaries are clear
- forbidden modifications are stated where necessary
- output expectations are clear
- validation expectations are clear
- referenced files and tools exist
- the agent does not duplicate detailed repository instructions
- broad discovery is justified by the agent role

Do not claim that an agent tool is supported or active unless the workspace
provides evidence.

#### 12. Validate VS Code Copilot Discovery

This skill runs inside VS Code Copilot Chat.

Validate from workspace evidence when possible:

- `.github/skills/copilot-context-auditor/SKILL.md` exists
- this skill contains valid YAML frontmatter
- the skill name is `copilot-context-auditor`
- the description narrowly identifies relevant audit and cleanup requests
- referenced instruction, prompt, agent, skill, script, and documentation paths
  exist
- `.instructions.md` files contain valid `applyTo` scope where applicable
- repository-wide instructions contain repository-wide guidance
- Markdown links resolve from the file containing each link

The user can use `/skills` in VS Code Copilot Chat to inspect configured skills.

Do not claim `/skills` was opened or that a skill appeared there unless that
action and result are available in the current session.

Do not use or recommend GitHub Copilot CLI commands as audit requirements.

If discovery cannot be verified, report:

`Not verified through the current VS Code Copilot Chat session.`

#### 13. Classify Every Finding

Use exactly one classification:

- `KEEP`
- `SHORTEN`
- `MOVE`
- `PATH-SCOPE`
- `TASK-SCOPE`
- `DEDUPLICATE`
- `FIX-CONFLICT`
- `FIX-REFERENCE`
- `ARCHIVE`
- `DELETE`
- `NOT-VERIFIED`

Use `DELETE` only when content is demonstrably redundant, stale, invalid, or
unused and no unique required guidance would be lost.

Use `KEEP` for reviewed content that is correctly owned and scoped.

#### 14. Prioritize Every Actionable Finding

Use exactly one priority:

- `P0`: conflicting, unsafe, invalid, broken, or capable of causing incorrect
  behavior
- `P1`: broadly injected duplication or substantial unnecessary always-on
  context
- `P2`: weak scoping, excessive examples, or repeated low-value text
- `P3`: naming, organization, or minor cleanup

Do not assign priority based only on file size.

`KEEP` findings do not require a priority unless they document an important
reason not to change something.

## Preferred Ownership Model

Use this model as a review framework, not as permission to reorganize files
without evidence.

### `.github/copilot-instructions.md`

Use for:

- minimal repository identity
- universally applicable hard rules
- universally applicable safety rules
- concise routing or discovery guidance

Avoid:

- large examples
- task-specific implementation details
- language-specific detail
- test-only detail
- deployment-only detail
- troubleshooting runbooks
- duplicated detailed documentation

### `.github/instructions/*.instructions.md`

Use for:

- path-scoped rules
- language-specific conventions
- framework-specific conventions
- artifact-specific rules
- test-specific rules
- infrastructure-specific rules
- validation relevant to matching files

Each file should have scope that matches its actual purpose.

### `.github/skills/*/SKILL.md`

Use for:

- reusable multi-step workflows
- tasks requiring judgment
- workflows involving multiple file inspections
- reusable validation processes
- specialized procedures

Descriptions should be narrow enough to avoid unrelated activation.

### `.github/prompts/*.prompt.md`

Use for:

- manually invoked one-shot tasks
- repeatable prompts
- tasks that do not need a persistent role
- tasks that should not always enter context

Do not place universally required safety rules only in a manually invoked
prompt.

### `.github/agents/*.agent.md`

Use for:

- recurring specialized roles
- stable tool policies
- explicit modification permissions
- role-specific validation expectations

Agents should have the narrowest tools required for their role.

### `AGENTS.md`

Use when:

- the repository intentionally supports multiple compatible coding agents
- guidance applies to the directory tree controlled by the file
- duplication with Copilot-specific instructions is avoided or explicitly
  justified

Do not assume nested behavior or precedence without workspace evidence.

### `docs/**/*.md`

Use for:

- detailed durable knowledge
- architecture explanations
- examples
- troubleshooting details
- operational runbooks
- deployment explanations
- local-development guidance
- testing strategy
- supporting evidence

Documentation does not automatically become Copilot context merely because it
exists.

### `scripts/**/*`

Use for:

- deterministic discovery
- deterministic validation
- repeated mechanical operations
- link validation
- frontmatter validation
- route or structure inventory
- checks that do not require model judgment

Scripts must use safe defaults and must not mutate production systems.

## Audit Deliverable

Create or update:

`docs/ai/copilot-context-audit.md`

Use this exact structure:

# Copilot Context Audit

## Executive Summary

Include:

- verified major findings
- highest-priority corrections
- unresolved verification limits
- statement that no fabricated token or credit estimates were used

## Scope

Include:

- workspace root
- operating mode
- included locations
- excluded locations
- limitations

## Repository State

Include:

- current branch, when available
- existing working-tree changes
- whether an earlier audit existed

## Inventory

For every discovered customization file include:

- path
- type
- activation
- scope
- references
- referenced by
- line count
- byte count
- audit disposition

## Context Graph

Show which files can cause or request other files to be inspected.

Label each relationship as:

- automatic
- conditional
- relevance-based
- manually invoked
- explicitly referenced
- documentation-only
- not verified

## Findings

For each finding use:

### AUDIT-###

- Priority:
- Classification:
- Source:
- Related sources:
- Activation scope:
- Evidence:
- Why it wastes context or creates risk:
- Exact recommended change:
- Expected behavioral impact:
- Validation:
- Status:

Finding IDs must remain stable when the audit is rerun.

Do not reuse an existing finding ID for a different issue.

## Duplicate Rules

List exact source and duplicate quotations.

## Conflicts

List exact conflicting quotations and required precedence.

If precedence is not proven, mark it for manual review.

## Broken or Unverified References

List:

- missing paths
- unresolved links
- unsupported activation assumptions
- unavailable referenced tools
- unresolved placeholders

## Recommended Target Structure

Show the proposed customization tree and responsibility of each file.

Do not include speculative files that are not required by a finding.

## Ordered Change Plan

List changes in dependency-safe order.

Map every change to one or more audit finding IDs.

## Skipped Recommendations

List possible optimizations rejected because:

- evidence was insufficient
- the change could weaken safety
- the change could change repository behavior
- activation could not be verified
- ownership could not be proven

## Validation Commands

List exact commands that can validate:

- paths
- links
- frontmatter
- references
- duplicate text
- Git diff
- repository state

Do not list unavailable or unverified commands as completed validation.

## Final Summary

Include mechanically determined counts for:

- files inspected
- findings by priority
- files proposed for modification
- files proposed for creation
- files proposed for movement
- files proposed for deletion
- items not verified

Do not calculate totals manually when they can be obtained mechanically.

## Cleanup Mode

Cleanup mode is enabled only when the user explicitly asks to apply, implement,
perform, or execute cleanup based on an existing audit.

An existing `docs/ai/copilot-context-audit.md` is required.

If the audit file does not exist, do not modify customization files. Run audit
mode instead and create the audit.

### Cleanup Input

The cleanup source of truth is:

`docs/ai/copilot-context-audit.md`

The current workspace remains authoritative.

Every audit finding must be revalidated before its recommendation is applied.

If current workspace evidence differs from the audit, use the current workspace
and record the mismatch.

### Cleanup Boundaries

Cleanup mode may modify only:

- `.github/copilot-instructions.md`
- `.github/instructions/**/*.instructions.md`
- `.github/copilot/**/*.md`
- `.github/skills/**`
- `.github/prompts/**/*.prompt.md`
- `.github/agents/**/*.agent.md`
- `.github/chatmodes/**/*.chatmode.md`
- `AGENTS.md`
- `**/AGENTS.md`
- `CLAUDE.md`
- `**/CLAUDE.md`
- `docs/**/*.md`
- scripts created specifically for Copilot customization validation

Cleanup mode must not modify:

- application source code
- application tests
- package or dependency definitions
- lock files
- infrastructure
- deployment configuration
- CI/CD pipelines
- database scripts
- production configuration
- secrets
- credentials
- generated output

### Required Cleanup Process

#### 1. Read the Audit

Read the complete audit before making any change.

Do not rely only on the executive summary or ordered change plan.

#### 2. Revalidate Findings

For every proposed change:

- locate the finding ID
- locate the quoted evidence
- verify the current source file exists
- verify the quoted text still exists
- verify the activation scope still matches
- verify the proposed destination exists or is justified
- verify the change remains safe
- verify the change remains within cleanup boundaries

If any requirement fails, skip the finding and record the exact reason.

#### 3. Establish Change Safety

Before editing:

- inspect Git status
- preserve all existing user changes
- do not discard or overwrite unrelated modifications
- identify files already modified before cleanup
- identify dependencies between proposed operations
- establish the ordered file-operation plan

Do not require a clean working tree, but record existing changes so cleanup
changes are distinguishable.

#### 4. Produce the File Operation Plan

Before editing, produce the exact ordered list of planned operations.

For every operation include:

- operation type
- source path
- destination path, if applicable
- audit finding IDs
- content being preserved
- validation required

Allowed operation types:

- create
- modify
- move
- delete

Then perform all safe operations without asking for confirmation.

#### 5. Apply Changes in Priority Order

Apply changes in this order:

1. P0 `FIX-CONFLICT`
2. P0 `FIX-REFERENCE`
3. P1 `DEDUPLICATE`
4. P1 `PATH-SCOPE`
5. P1 `TASK-SCOPE`
6. P1 `MOVE`
7. P1 `SHORTEN`
8. P2 changes directly supported by evidence
9. P3 changes only when required by another applied change

Do not apply a lower-priority change when it depends on a skipped
higher-priority change.

#### 6. Preserve Rule Strength

When moving or consolidating content:

- preserve normative strength
- preserve exceptions
- preserve examples that define required behavior
- preserve validation commands
- preserve safety conditions
- preserve repository-specific terminology
- preserve referenced paths when still valid
- preserve the narrowest proven activation scope

Do not paraphrase a hard rule if paraphrasing could change its meaning.

#### 7. Apply the Ownership Model

When supported by audit evidence:

- keep global rules in `.github/copilot-instructions.md`
- move path-specific rules to `.instructions.md`
- keep multi-step workflows in skills
- keep one-shot manual tasks in prompts
- keep recurring roles in agents
- keep detailed explanation and examples in documentation
- keep deterministic checks in scripts
- use concise references instead of copied detailed text

Do not create new files merely to satisfy this model if the existing structure
is already correctly scoped.

#### 8. Update References

When moving, renaming, or deleting a file:

- update all verified incoming references
- update relative Markdown links
- update routing guidance
- update indexes
- update prompt, skill, and agent references
- search the workspace for the old path
- verify no required reference remains unresolved

Do not replace a broken reference with a guessed path.

#### 9. Safe Deletion Rules

Delete a customization file only when all conditions are true:

1. The audit identifies it as redundant, stale, invalid, or unused.
2. Current workspace evidence confirms the finding.
3. Every unique required rule has a verified destination.
4. All incoming references have been updated or removed.
5. No remaining instruction, skill, prompt, agent, AGENTS.md, CLAUDE.md, or
   documentation file requires it.
6. Deletion does not weaken safety, architecture, validation, or repository
   behavior.
7. The cleanup report records the deletion and its evidence.

If any condition cannot be verified, do not delete the file.

Use `ARCHIVE` only when the repository already has a verified archival
convention. Do not invent an archive location.

#### 10. Validate Cleanup

After editing:

1. Validate YAML frontmatter.
2. Validate every `applyTo` pattern syntactically.
3. Validate all Markdown links.
4. Validate all referenced paths.
5. Search for references to moved, renamed, or deleted files.
6. Search for duplicate copies of rules changed during cleanup.
7. Confirm `.github/copilot-instructions.md` contains only globally applicable
   guidance.
8. Confirm skill descriptions are narrowly triggered.
9. Confirm prompt files remain manually or explicitly invoked.
10. Confirm agent roles and tool scopes remain bounded.
11. Confirm no security rule was removed or weakened.
12. Confirm no production-safety rule was removed or weakened.
13. Confirm no architecture rule was removed or weakened.
14. Confirm no testing or coverage requirement was removed or weakened.
15. Confirm no application source code was modified.
16. Confirm no out-of-scope file was modified.
17. Review Git diff for unintended changes.

Do not claim a validation passed unless it was performed successfully.

#### 11. Rerun the Audit

After cleanup, rerun the audit process against the resulting workspace.

Update existing finding IDs:

- `Resolved`
- `Partially resolved`
- `Skipped`
- `Still open`
- `Not verified`

Do not delete finding history from the audit report.

Add new finding IDs only for newly discovered issues.

#### 12. Record Cleanup Results

Update:

`docs/ai/copilot-context-audit.md`

Create or update:

`docs/ai/copilot-context-cleanup.md`

### Cleanup Report Structure

Use this exact structure:

# Copilot Context Cleanup

## Scope

Include:

- workspace root
- source audit path
- cleanup boundaries
- repository-state limitations

## Preexisting Changes

List files that were already modified before cleanup.

## Planned Operations

List the exact ordered file-operation plan and mapped audit finding IDs.

## Applied Changes

For every applied change include:

### CLEANUP-###

- Audit finding IDs:
- Operation:
- Source:
- Destination:
- Evidence:
- Exact change:
- Content preserved:
- Why the change reduces unnecessary context:
- Activation after cleanup:
- Validation:
- Status:

## Files Created

List each path and associated cleanup IDs.

## Files Modified

List each path and associated cleanup IDs.

## Files Moved

List source, destination, and associated cleanup IDs.

## Files Deleted

List each path, associated cleanup IDs, and deletion-safety evidence.

## Findings Resolved

List audit finding IDs and resolution status.

## Findings Skipped

For every skipped finding include:

- finding ID
- exact reason
- failed prerequisite
- manual review required

## Validation Results

List:

- validation performed
- command or method
- result
- failures
- unverified checks

## Remaining Risks

List unresolved conflicts, unverified behavior, and manual-review requirements.

## Final Repository Structure

Show the resulting Copilot customization tree.

## Cleanup Completion Rules

Do not claim cleanup is complete if:

- required validation failed
- a moved or deleted path is still referenced
- a hard rule was lost
- an out-of-scope file was modified
- P0 findings remain unresolved without being reported
- audit findings were applied without current evidence
- the post-cleanup audit was not completed

If validation fails:

- do not hide the failure
- do not fabricate a successful result
- preserve the current evidence
- record the exact failure
- identify affected files
- identify the safest next action

## Invocation Guidance

### Audit-Only Invocation

Use this skill in a new VS Code Copilot Chat session with:

`/copilot-context-auditor`

Then request:

- audit this workspace
- audit-only mode
- do not modify existing files
- create `docs/ai/copilot-context-audit.md`

### Cleanup Invocation

Start a new VS Code Copilot Chat session after reviewing the audit.

Use:

`/copilot-context-auditor`

Then explicitly request:

- apply cleanup recommendations from
  `docs/ai/copilot-context-audit.md`
- cleanup mode
- modify only allowed customization files
- revalidate every finding
- create `docs/ai/copilot-context-cleanup.md`
- rerun the audit after cleanup

Do not continue cleanup in a long audit conversation when a fresh chat is
available. The audit file is the durable cleanup input.

## Final Response Contract

### Audit Mode

The final response must contain only:

- Audit report path
- Files inspected
- Findings by priority
- P0 findings
- P1 findings
- Items not verified
- Existing files modified
- Validation results

`Existing files modified` must be `None` unless the audit report already existed
and was updated.

### Cleanup Mode

The final response must contain only:

- Files created
- Files modified
- Files moved
- Files deleted
- Findings resolved
- Findings partially resolved
- Findings skipped
- Validation results
- Remaining manual review

Do not include progress narration, generic advice, speculative savings, or
unsupported completion claims.
