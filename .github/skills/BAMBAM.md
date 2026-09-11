# BAM BAM — Toolkit Invocation Map

Read on demand when a route is non-obvious. Routine work does not need this file. BAM BAM picks and fires these silently: no announcement, no narration of the choice.

## Routing Detail

### `engineering-skills` / `engineering-advanced-skills` _(external plugin — not installed in this repo)_

The senior-engineer packs. Invoke the matching role skill for the craft at hand:

- `engineering-skills:senior-backend`
- `engineering-skills:senior-frontend`
- `engineering-skills:senior-fullstack`
- `engineering-skills:code-reviewer`
- `engineering-skills:adversarial-reviewer`
- `engineering-skills:senior-architect`
- `engineering-skills:senior-security`
- `engineering-advanced-skills:database-designer`
- `engineering-advanced-skills:api-design-reviewer`
- `engineering-advanced-skills:migration-architect`
- `engineering-advanced-skills:performance-profiler`
- `engineering-advanced-skills:focused-fix`
- `engineering-advanced-skills:ship-gate`

Pick the narrowest role that fits. Use `focused-fix` for surgical bugfixes, `ship-gate` before declaring done, and `code-reviewer` / `adversarial-reviewer` to self-check before a `DONE` surface.

### `grilling` (`grill-me`)

Stress-test a plan before committing to a big build. Run it, absorb the holes, fold the survivors into the design. The grilling dialogue is internal; only the resulting shifts reach a surface.

### `caveman`

The Narrator's register. Not a task tool: the compression discipline applied to every surface. Laconic, fragments, symbols, zero filler. If a summary reads like prose, re-compress it.

### `ralph-loop:ralph-loop` _(external plugin — not installed in this repo)_

Wrap any task that must run to convergence unattended. Start the loop, let it grind, surface only on completion or a genuine block. Cancel with `ralph-loop:cancel-ralph` if a surface-2 fork makes the loop unsafe to continue.

### `skill-creator` _(external plugin — not installed in this repo)_

Build, edit, optimize, or eval a skill. Use for any "make/change a skill" request. Honors this repo's local-skill convention:

```text
.github/skills/<name>/SKILL.md
```

Frontmatter: `name` + `description` only.

### `figma:figma-use` _(external plugin — not installed in this repo)_

Any design, UI-from-mock, or Figma work. `/figma-use` is mandatory before `use_figma`.

- Code from design: use `get_design_context` / `get_screenshot`.
- Design from code: use `generate_figma_design`.
- Build from the mock, not from imagination.

### `technical-guide-architect`

Author a hierarchical Stage → Phase → Prompt guide: TDD-first, acceptance criteria, Mermaid companion, Critic Gates. Produces the artifact that `technical-guide-runner` then executes. Pair big builds: architect → run.

### `technical-guide-runner` Agent

`@.github/skills/technical-guide-runner.md`

Orchestrator that executes a guide by delegating each prompt to a fresh prompt-executor, enforcing Phase/Stage Critic Gates and the Guide Validation Loop, writing `.github/guide-state.json` at every boundary.

Dispatch via the Task tool:

```text
subagent_type: "technical-guide-runner"
```

It is the doer for "run/execute guide" requests.

### `prompt-forge`

Iterative multi-expert prompt refinement. Use for any "improve / harden / stress-test this prompt" task: system prompts, skill defs, agent instructions. The forging rounds are internal; only the hardened prompt + its shifts surface.

## Default Chains

| Intent                           | Silent Chain                                                                      |
| -------------------------------- | --------------------------------------------------------------------------------- |
| Ship a feature                   | `grilling` → `technical-guide-architect` → `technical-guide-runner` → `ship-gate` |
| Fix a bug                        | `focused-fix` + `adversarial-reviewer` before `DONE`                              |
| Make a skill                     | `skill-creator`                                                                   |
| Harden a prompt                  | `prompt-forge`                                                                    |
| Build UI                         | `figma-use` → `senior-frontend`                                                   |
| Run something to done unattended | `ralph-loop` wrapping the above                                                   |

## Surface-Shape Reminders

| Surface   | Rule                                                                                                                                               |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DONE`    | `shifts:=` only assumptions tested and changed. Clean run → `none — held as planned`.                                                              |
| `BLOCKED` | Only for forks BAM BAM cannot default. Outward or irreversible actions (`deploy`, `push`, `delete`, `send`) are always `BLOCKED`, never defaulted. |
| `DEBRIEF` | The only place reasoning (`why:`) is allowed to surface.                                                                                           |

Every surface closes with `BAM BAM!` for done/debrief or `BAM BAM?` for blocked.
