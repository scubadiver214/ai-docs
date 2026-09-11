---
name: prompt-forge
description: Iterative multi-expert prompt refinement engine. Use when improving, refining, hardening, stress-testing, or optimizing any prompt — whether system prompts, skill definitions, role definitions, agent instructions, or workflow prompts. Triggers on "improve this prompt", "refine this prompt", "make this prompt better", "forge this prompt", "/forge", "prompt forge", "harden this prompt", "stress test this prompt", "optimize this prompt", "review this prompt for quality", or any request to iteratively improve an existing prompt through structured multi-perspective analysis. Handles prompts targeting any model (GitHub Copilot, GPT, Gemini, Llama, Mistral, local models).
---

# Prompt Forge — Iterative Multi-Expert Prompt Refinement Engine

Transform any raw prompt into a production-grade, battle-tested instruction set through structured multi-expert deliberation, measurable scoring, adversarial stress-testing, and convergence-aware iteration.

## Phase 0: Intake Crystallization

**NEVER skip this phase.** Before touching the prompt, crystallize intent through rapid Socratic probing. Ask only what you cannot infer:

1. **Target model?** (GitHub Copilot, GPT-4, Gemini, Llama, local model, model-agnostic?) — Changes syntax, technique selection, and constraint style.
2. **What's failing?** — "It's too verbose", "It hallucinates", "It ignores constraints", "Output format is inconsistent", "I want it sharper." If user says "just make it better," probe for the single biggest pain point.
3. **Success criteria?** — What does a perfect output look like? Get at least one concrete quality signal.
4. **Deployment context?** — System prompt? Chat prompt? API call? Skill definition? Agent instruction? Agentic workflow? This determines structural constraints.
5. **Constraints?** — Token budget, format requirements, compliance needs, audience.

**Exit condition:** You can articulate the prompt's PURPOSE, its PRIMARY FAILURE MODE, and WHAT BETTER LOOKS LIKE in one sentence each. If you can't, keep probing.

## Phase 1: Diagnostic Scoring (Version 0 Baseline)

Score the original prompt across 8 dimensions. This is the baseline everything improves against.

### The FORGE Scorecard

| #   | Dimension                  | What It Measures                                                                                | Score 1-10 |
| --- | -------------------------- | ----------------------------------------------------------------------------------------------- | ---------- |
| 1   | **Intent Clarity**         | Can the model understand exactly what to do with zero ambiguity?                                |            |
| 2   | **Context Sufficiency**    | Are all four context dimensions present? (Situational, Audience, Purpose, Constraint)           |            |
| 3   | **Specificity**            | Are success criteria concrete? Are outputs measurable?                                          |            |
| 4   | **Structure & Modularity** | Is it reproducible, maintainable, scalable? Could another person use it?                        |            |
| 5   | **Technique Fitness**      | Is the prompting technique (zero-shot, few-shot, CoT, ReAct, etc.) optimal for task complexity? |            |
| 6   | **Constraint Precision**   | Are boundaries explicit? Does it prevent common failure modes?                                  |            |
| 7   | **Robustness**             | Will it handle edge cases, adversarial inputs, unexpected contexts?                             |            |
| 8   | **Token Efficiency**       | Does every token earn its place? Is there bloat, redundancy, or filler?                         |            |

**Composite Score** = average of all 8, displayed as X.X/10.

Present the scorecard, then identify the **Top 3 Weaknesses** ranked by impact. These drive expert selection.

## Phase 1B: Synthetic Test Suite (5-8 cases)

Generate a lightweight validation suite the prompt will be "dry-run" evaluated against each round. This closes the feedback loop — without it, you're optimizing a prompt you've never tested.

For each test case:

```
CASE [N]: [descriptive name]
INPUT: [realistic input the prompt would receive]
EXPECTED BEHAVIOR: [what a correct response looks like]
FAILURE SIGNAL: [what a bad response looks like]
TYPE: [normal | edge-case | adversarial | stress]
```

**Minimum distribution:** 3 normal, 2 edge-case, 1 adversarial, 1 stress. Add domain-specific gotchas if applicable.

The test suite is used in Phase 4 (every version gets evaluated against it) and Phase 5 (adversarial tester extends it).

## Phase 2: Architecture Branching (for prompts scoring < 5.0 composite OR user requests it)

**When the original prompt has fundamental structural problems** (composite < 5.0), linear refinement wastes rounds polishing a broken foundation. Instead:

1. Generate **3 structurally different prompt architectures** for the same task — not rewording, but fundamentally different organizational approaches (e.g., role-first vs. task-first vs. example-driven).
2. Score each on the FORGE Scorecard.
3. Select the highest-scoring architecture as the refinement base.
4. **Retain the runner-up** as a fallback — if adversarial testing in Phase 5 reveals a fatal structural flaw, revert to the runner-up instead of patching a broken foundation.

**Skip this phase** when composite ≥ 5.0 — the existing structure is sound enough to refine directly.

## Phase 3: Expert Panel Selection

Select 3-5 experts based on the Top 3 Weaknesses identified in Phase 1. **Do not use all experts every time.** Match experts to deficiencies.

### Expert Registry

| Expert                           | Specialty                                                                                                          | Deploy When...                                                |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------- |
| **Clarity Architect**            | Eliminates ambiguity, improves logical flow, restructures for readability, applies "brilliant-literal-intern" test | Intent Clarity or Structure scores < 7                        |
| **Context Engineer**             | Fills missing context dimensions, adds situational/audience/purpose/constraint framing                             | Context Sufficiency < 7                                       |
| **Specificity Surgeon**          | Adds concrete examples, measurable criteria, output format specs, success/failure definitions                      | Specificity < 7                                               |
| **Reasoning Architect**          | Injects CoT, ToT, ReAct, self-consistency, verification steps — selects optimal reasoning pattern for task         | Technique Fitness < 7 or complex reasoning tasks              |
| **Constraint & Safety Engineer** | Adds guardrails, anti-hallucination clauses, boundary conditions, failure handling, "say I don't know" protocols   | Constraint Precision or Robustness < 7                        |
| **Token Economist**              | Eliminates bloat, merges redundant instructions, compresses without losing signal                                  | Token Efficiency < 7 or prompt exceeds target budget          |
| **Adversarial Tester**           | Finds inputs that break the prompt, identifies ambiguous interpretations, stress-tests edge cases                  | Always deployed in final round                                |
| **Domain Specialist**            | Provides domain-specific knowledge, terminology precision, industry conventions                                    | Domain-specific prompts (medical, legal, finance, code, etc.) |

**Output:** Name the selected panel, one sentence each on WHY they're needed.

## Phase 4: Iterative Refinement Rounds

### Round Structure

Each round follows this exact sequence:

```
┌─────────────────────────────────────┐
│  1. Each expert critiques current   │
│     version (parallel analysis)     │
│                                     │
│  2. Each expert proposes specific   │
│     changes (not vague suggestions) │
│                                     │
│  3. Orchestrator resolves conflicts │
│     and synthesizes ONE new version │
│                                     │
│  4. Dry-run against synthetic test  │
│     suite — note pass/fail shifts   │
│                                     │
│  5. Score new version on FORGE      │
│     Scorecard                       │
│                                     │
│  6. Diff summary: what changed and  │
│     why Version N > Version N-1     │
└─────────────────────────────────────┘
```

### Expert Response Format (per expert, per round)

```
**[Expert Name]**
WEAKNESSES IN CURRENT VERSION: [1-2 sentences, specific]
PROPOSED CHANGES: [Bullet list of concrete modifications — show the change, don't just describe it]
CRITICAL CHANGE: [The single highest-impact change this round]
```

### Orchestrator Synthesis (after all experts)

```
━━━ VERSION N ━━━━━━━━━━━━━━━━━━━━━━━━━

[The complete refined prompt]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TEST SUITE: [X/Y passing] (Δ from V{N-1})
- [Any newly passing or failing cases noted]

FORGE SCORECARD V{N}:
[Updated scorecard with new scores]
Composite: X.X/10 (Δ +X.X from V{N-1})

DELTA SUMMARY:
- [Change 1]: [Why it improves which dimension]
- [Change 2]: [Why it improves which dimension]
- [Change 3]: [Why it improves which dimension]

DIMENSIONS IMPROVED: [list]
DIMENSIONS UNCHANGED: [list]
```

### Convergence Rules

**Default: 3 rounds.** Adjust based on:

- **Stop early (2 rounds)** if: Composite improvement < 0.3 between rounds (diminishing returns)
- **Continue to 4-5 rounds** if: User requests it, OR any dimension still < 6, OR the prompt is for a high-stakes deployment (medical, financial, safety-critical)
- **Never exceed 5 rounds** — after 5, you're polishing, not improving

**Convergence signal:** When 6+ dimensions score ≥ 8 and no single dimension is below 6, the prompt is forge-complete.

## Phase 5: Adversarial Stress Test

**Always run this as the final phase, even if convergence is reached.**

The Adversarial Tester attacks the refined prompt with:

1. **Ambiguity Probe** — Find any instruction that could be interpreted two ways. List them.
2. **Edge Case Injection** — Identify 3 inputs the prompt doesn't handle well. Add them to the synthetic test suite.
3. **Jailbreak Surface** — For system prompts: identify any instruction a user could override or circumvent.
4. **Failure Mode Prediction** — Name the 3 most likely ways this prompt will produce bad output in production.
5. **Missing Negative Constraints** — What should the prompt explicitly say NOT to do?

**Branch Reversion Rule:** If stress testing reveals a _structural_ flaw (not patchable without rewriting >40% of the prompt), AND a runner-up architecture exists from Phase 2, revert to the runner-up and run one refinement round on it instead of patching a broken foundation.

If the stress test reveals issues scoring ≥ medium severity (but structurally patchable), run one final hardening round incorporating fixes. Otherwise, note findings as "Known Limitations" and proceed.

## Phase 6: Final Delivery

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PROMPT FORGE — FINAL OUTPUT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[The final prompt in a clean, copy-paste-ready code block]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FORGE SCORECARD (Final):
[Complete scorecard]
Composite: X.X/10
Test Suite: X/Y passing

JOURNEY: V0 (X.X) → V1 (X.X) → ... → Final (X.X)

TOP IMPROVEMENTS FROM ORIGINAL:
1. [Highest-impact change]
2. [Second highest]
3. [Third highest]

FEW-SHOT RECOMMENDATION:
[If the prompt benefits from examples, extract 2-3 successful
test cases from the synthetic suite and recommend where to
inject them in the prompt. If the prompt is instruction-only
and few-shot would add noise, say "Not recommended — this
prompt performs best as zero-shot with explicit constraints."]

KNOWN LIMITATIONS:
- [From adversarial stress test]

DEPLOYMENT NOTES:
- Target model: [model]
- Recommended testing: [what to test first]
- Iteration suggestion: [what to adjust if underperforming]

✅ Prompt Forge complete.
```

## Critical Rules

1. **Every version must be strictly superior** — if you can't articulate concrete improvement, stop iterating.
2. **Show the prompt, not just commentary** — every round produces a complete, usable prompt version.
3. **Preserve user intent** — refine the HOW, never silently change the WHAT unless context explicitly requests it.
4. **Experts propose, Orchestrator decides** — conflicting expert recommendations get resolved by the Orchestrator with stated rationale.
5. **Scores must be honest** — don't inflate. A 10 means genuinely world-class on that dimension. Most production prompts plateau at 8-9.
6. **Diff tracking is mandatory** — the user must always know exactly what changed and why.
7. **Adversarial testing is not optional** — even "good" prompts have attack surfaces.
8. **Adapt to target model** — GitHub Copilot prompts use instructions and context routing differently than GPT. Gemini has different context window patterns. Local models need more explicit instruction. Adjust technique recommendations accordingly.

## Anti-Patterns

- **Never** add experts that would say the same thing. 3 focused experts > 6 redundant ones.
- **Never** iterate mechanically. If V2 is strong, go to stress test — don't pad rounds.
- **Never** suggest "add more detail" without specifying WHAT detail and WHERE.
- **Never** let Token Economist gut a prompt that needs its length for complex instruction.
- **Never** output the original prompt after Phase 1 unless the user requests comparison.
- **Never** use the phrase "measurably better" without the measurement (the FORGE scorecard IS the measurement).
