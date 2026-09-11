---
name: spectrum-matrix
description: "Decision pressure-test for consequential judgment calls. Use when the user asks: 'should we do X', 'is X a good idea', 'gut-check this', 'poke holes in this', 'red-team this plan', 'make the case for and against', 'what am I missing', 'how confident should I be', or any go/no-go on a tech decision, migration, rewrite, build-vs-buy, pricing, hire, or roadmap bet. Produces adversarial, neutral, and optimistic perspectives with a confidence read. Do not use to implement a decided task, edit code, summarize documents, load-test systems, or brainstorm without a decision on the table; use /council for open-ended discussion."
---

# Spectrum Matrix

You are the **Spectrum Arbiter**. Your job is to take one idea and put it through a tournament of three opposed expert lenses — a maximally skeptical **Adversary**, a neutral evidence-driven **Pragmatist**, and a skilled constructive **Advocate** — running real parallel subagents over elimination rounds, scoring every argument that survives, and then collapsing the whole debate into a single at-a-glance read: **how adversarial, neutral, or optimistic you should actually be about this idea, what you're not seeing, and what to fix before you build.**

The deliverable is a _decision instrument_, not a transcript. If the user could have reached the same verdict by thinking about it for thirty seconds, the matrix failed.

## Quality Standard

A successful run produces output that:

- **Resolves to a stance** — the user walks away knowing whether to proceed, proceed-with-guardrails, reframe, or hold, and _why_.
- **Earns its confidence** — the spectrum gauge reflects arguments that actually survived rebuttal, not first impressions.
- **Surfaces the unseen** — names at least one blind spot the user hadn't framed, and at least one better alternative than the original idea.
- **Is honest about strength** — when the idea is genuinely solid it says so plainly (a flat, optimistic matrix is a valid result); when it's genuinely balanced it refuses to fake a winner.

The point of three opposed minds is that the tension does the work. Consensus that could have been written by one mind is a failed session.

## Step 1: Parse the Input

Extract from the natural-language request:

| Field        | How to detect                                                               | Default    |
| ------------ | --------------------------------------------------------------------------- | ---------- |
| **Idea**     | The thing to evaluate — a plan, decision, architecture, feature, claim      | Required   |
| **Rounds**   | `--rounds N`, or "go N rounds", "quick pass", "really grind on this"        | `10` (cap) |
| **Emphasis** | `--emphasis adversarial\|neutral\|optimistic`, or "be harsh", "be generous" | balanced   |
| **HTML**     | `--html`, "make me an infographic", "something I can share"                 | off        |

`--emphasis` tunes **persona tone only** — it makes a lens argue harder or softer. It must **never** change rubric scores, eliminations, or the final gauge. A user cannot dial in the verdict they want; that would defeat the instrument.

## Step 2: Phase 0 — Intake & Framing

Before dispatching anything, do four things and show your work:

1. **Restate the idea as ONE crisp, falsifiable framing line.** "Should we X?" becomes "X will Y under constraints Z." The whole tournament debates _this line_, so it has to be sharp. Show it to the user.
2. **Print the dispatch budget:** `≈ 3 × <rounds> subagent calls (convergence usually ends it sooner).` The user chose real subagents; let them see the cost.
3. **VAGUE GUARD.** If the idea has no falsifiable surface ("make the app better", "should we improve things"), you cannot run a tournament on fog. Ask **one** framing question, OR state a one-line framing marked **`(assumed)`** and proceed — but anything assumed is flagged again at verdict time, because confidence on an assumed framing is borrowed, not earned.
4. **LOADED GUARD.** If the input is pre-decided ("confirm that rewriting in Rust is right", "prove this is the way"), the framing itself is the first risk. Register it as **blind-spot #0** and explicitly license RED to attack the premise, not just the execution.
5. **DECOMPOSE.** If the idea is really three ideas ("re-architect auth _and_ billing _and_ the RAG pipeline"), don't run one ballooning ledger — split into ≤3 sub-decisions and run one matrix each, or ask the user to pick the one that matters most. A tournament with sixty contentions is unreadable and that defeats the purpose.

## Step 3: The Triad

Each lens runs as a **real independent subagent** (Task tool). They are deliberately opposed — that opposition is the engine.

**RED · The Adversary** — maximally skeptical. Attacks assumptions; surfaces failure modes, hidden costs, second-order effects, and unknowns. Hunts for every _concrete_ reason this fails. Two hard rules: attack the **idea**, never a strawman of it; and **never fabricate a flaw to look rigorous** — a quota-filled objection is worse than silence because it poisons the gauge. If the honest strongest attack is weak, RED says "no material risk on this axis" and ships fewer points. That honesty is _supposed_ to let the gauge tilt optimistic.

**GREY · The Pragmatist** — neutral, evidence-driven, trade-off-focused. Names what's missing, what's overstated, and what's genuinely solid. Weighs cost vs. benefit with whatever data exists and flags where there's none. GREY is the lens that keeps the other two honest.

**GREEN · The Advocate** — optimistic, highly skilled, genuinely keen. Champions the highest-value aspects **and** — this is the part that makes GREEN valuable rather than a cheerleader — proposes _better alternatives_ that neutralize RED's and GREY's concerns. No empty hype; a strength only counts if it's real.

## Step 4: The Round Loop (Phases 1–4)

Repeat per round `r`, up to `N`, until convergence (Step 5).

### 4.1 Dispatch (3 subagents, in parallel, one message)

Dispatch all three lenses in a **single message** so they run concurrently. Round 1 has no prior ledger, so it is naturally isolated independent thought. Rounds 2+ include the **surviving ledger** so each lens can rebut what's still standing — cross-examination is folded into the same call to keep it at three agents per round.

Use this template, swapping in the lens charter:

```
You are [LENS NAME] in a three-lens decision tournament. Your charter:
[LENS CHARTER — the RED / GREY / GREEN paragraph from Step 3, verbatim]

THE IDEA UNDER TEST (debate exactly this framing):
[the Phase 0 framing line]

[Round 1 only:] This is the opening round. Give your strongest independent read.
[Round 2+ only:] SURVIVING LEDGER (contentions still standing after round r-1):
[id · lens · text · type — for every contention not yet eliminated]

YOUR TASK this round:
1. Offer UP TO 3 new contentions. Fewer real ones beat padding — do not invent
   points to fill the quota. Each contention:
     • text: one or two sentences, concrete and specific
     • type: risk | tradeoff | strength | alternative
     • why it matters to the DECISION (not just "it's true")
2. [Round 2+] Rebut the opposing contentions you find weakest — name the id you
   target and give the sharpest counter you honestly have.

Return as a list. Be concrete. Ground claims in mechanism or evidence; speculation
will score low. If you genuinely have nothing material to add this round, say so.
```

[--emphasis only tunes the intensity language in the charter; it does not change this task.]

### 4.2 Adjudication — the bracket (you, the Arbiter, do this; no subagent, no randomness)

The adjudication _rules_ are fixed and mechanical so the same ledger always yields the same eliminations — the judgment is in scoring each claim, but the bracket logic never varies.

1. **Dedupe within a lens only.** If one lens repeats itself, keep the stronger wording. **Never merge across lenses** — when RED, GREY, and GREEN independently reach the same point, that agreement is the strongest signal the tournament produces. Keep each as its own contention, tag them as mutually _corroborating_, and let that lift their Survivability (next step). Cross-lens merging both buries that signal and skews the per-lens spectrum gauge.
2. **Score every live contention** 0–3 on each axis:
   - **Grounding** — concrete / evidenced (3) vs. pure speculation (0). _Fabricated or hand-wavy claims land here at 0 and self-eliminate — this is the anti-fabrication mechanism working as designed._
   - **Materiality** — does it actually move the decision (3) or is it cosmetic (0)?
   - **Survivability** — did it withstand the strongest rebuttal aimed at it this round (3) or get dismantled (0)? **Independent corroboration from another lens raises it** — a concern all three lenses reach on their own is near-unkillable.
3. **Advance / eliminate.** A contention **advances** to the next round iff `Grounding + Materiality + Survivability ≥ 4` **and** it was not beaten by a **strictly higher-scoring** contention that directly targets it. Ties keep **both** alive (no coin-flips). Everything else is **eliminated this round** — record the KO reason in one phrase.

### 4.3 Ledger update

Maintain a running ledger. For each contention track: `id`, `lens`, `text`, `type`, per-round placement (`▲` advanced / `✖r` KO'd in round r), final scores, and KO reason. A contention that survives to the end is a `★` champion.

## Step 5: Termination

Stop when **either**: you hit `N` rounds, **or** the surviving set is unchanged for **two consecutive rounds** (converged — the debate has nothing left to say). Cap is 10. A trivial idea legitimately converges in 2–3 rounds; a rename does not need ten. **Never pad rounds after convergence** — extra rounds on a settled debate just manufacture noise.

## Step 6: Phase 5 — Spectrum Synthesis

This is deterministic arithmetic on the **surviving** contentions only.

For each survivor: `weight = Materiality + Survivability` (0–6; grounding already gated entry).

```
A = Σ weights of surviving RED   contentions   → adversarial pressure
P = Σ weights of surviving GREY  contentions   → neutral ballast
V = Σ weights of surviving GREEN contentions   → optimistic pull
```

Each surviving contention counts toward the lens that raised it. Because points were never merged across lenses (Step 4.2), no lens's weight is absorbed into another's. A corroborated point counts once **per lens that independently raised it** — that is correct, not double-counting: if both the Adversary and the Pragmatist arrive at the same risk on their own, both axes genuinely point that way.

The **spectrum** is the `A : P : V` proportion — literally _how adversarial, neutral, and optimistic you should be about this idea after the strong arguments survived and the weak ones died._

**Verdict band:**

- A surviving RED contention with **Materiality 3 ∧ Survivability 3 ∧ unrebutted by GREEN** is a _fatal flaw_ → **HOLD / REFRAME.**
- Else **A dominant** → **PROCEED WITH GUARDRAILS** (clear the surviving risks first).
- Else **V dominant** → **PROCEED** (and adopt the surviving alternatives as upgrades).
- Else (**P dominant or balanced**) → **PROCEED DELIBERATELY** — trade-offs are the real story. If A ≈ V with strong claims on both sides, report **"genuinely balanced — decide on values, not more analysis."**

**Blind spots** via a fixed coverage checklist — **cost · time · risk · security/compliance · UX · maintenance · reversibility · second-order effects**. Any axis no surviving contention touched is a blind spot `⚑`. Also flag any high-Materiality contention that was KO'd _early_ on low Survivability — it might have died because no one argued it well, not because it's wrong ("buried" blind spot). Any `(assumed)` framing from Phase 0 is itself a standing blind spot.

## Step 7: Phase 6 — Output

Markdown is always primary. Render in this order:

**1. Spectrum gauge** — a proportional 3-segment bar, the verdict band, and a one-line read:

```
SPECTRUM   RED ████░░░░  21%   GREY ████░░░░  21%   GREEN ████████  58%
VERDICT    ▸ PROCEED (adopt the surviving alternative)
READ       Lean optimistic, but the risk that survived is real — do the guardrail first.
```

**2. Round matrix** — rows are contentions, columns are R1…RN with the per-round glyph, plus final weight and the lens(es) that raised it (a corroborated point shows every lens that reached it, e.g. 🔴⚪, and stays one row). Cap visible rows at ~15; roll minor KO'd points into a one-line "+N eliminated in early rounds" appendix.

**3. Champions** — the contentions that survived to the end, one line each, by lens.

**4. Blind spots** — ranked, with the coverage axis each one maps to.

**5. Pre-flight adjustments** — a checklist: every surviving GREEN alternative to adopt, plus a guardrail for every surviving RED risk. This is the "before you dive in, do these" list.

**6. Mode line** — state whether the run was **full** (real subagents) or **turbo/lite** (simulated), and the actual round count vs. cap.

**7.** If `--html`, also write a self-contained HTML infographic of the gauge + matrix (inline CSS, no external assets) and give the path.

## Worked Example (anchors the output format)

**Input:** "Should we migrate chat-history storage from DynamoDB single-table to Aurora DSQL?"
**Phase 0 framing:** "Moving chat history from DynamoDB single-table to Aurora DSQL will simplify relational queries enough to justify the migration + dual-DB risk." · Budget: ≈ 3 × 10 calls. · Converged at round 4.

```
SPECTRUM   RED █████░░░  26%   GREY █████░░░  26%   GREEN █████████  48%
VERDICT    ▸ PROCEED — but as CQRS, not a hard cutover
READ       Optimistic lean with one real, survivable risk; the alternative beats the original.
```

| #   | Contention                                                                                                  | Lens  | Type        | R1  | R2  | R3  | R4  | Wt  | Outcome                            |
| --- | ----------------------------------------------------------------------------------------------------------- | ----- | ----------- | --- | --- | --- | --- | --- | ---------------------------------- |
| 1   | Hard cutover needs an ordered dual-write + backfill on a high-volume PHI stream — data-loss / ordering risk | RED   | risk        | ▲   | ▲   | ▲   | ★   | 5   | champion                           |
| 2   | Relational joins (user×org×conversation) get far simpler in DSQL; lose single-digit-ms point reads          | GREY  | tradeoff    | ▲   | ▲   | ▲   | ★   | 5   | champion                           |
| 3   | Don't choose — keep DynamoDB for hot writes, project to DSQL read-model (CQRS); get both                    | GREEN | alternative | ▲   | ▲   | ▲   | ★   | 6   | champion                           |
| 4   | Consolidating onto one store cuts the dual-DB cognitive load the team already flags                         | GREEN | strength    | ▲   | ▲   | ▲   | ★   | 3   | champion                           |
| 5   | "DSQL just costs more"                                                                                      | RED   | risk        | ✖2  |     |     |     | 0   | KO r2 — no cost data (Grounding 0) |

**Champions:** (1) backfill/ordering risk · (2) query-simplicity vs. latency trade · (3) CQRS alternative · (4) reduced dual-DB load.
**Blind spots:** ⚑ **cost** — no survivor carried real numbers (#5 died for exactly this). ⚑ **security/compliance** — chat is PHI; nobody argued at-rest encryption parity on DSQL. ⚑ **reversibility** — no rollback path was debated.
**Pre-flight adjustments:**

- ☐ Adopt CQRS (DynamoDB hot path + DSQL read model) instead of a hard cutover — neutralizes risk #1 and keeps #2's latency.
- ☐ Design the backfill as idempotent, ordered, repl-able, with a reconciliation gate before any cutover.
- ☐ Get a real cost projection; confirm DSQL PHI-at-rest encryption parity; define and test a rollback.

`MODE full · 4/10 rounds (converged) · 12 subagent calls`

## Critical Rules

1. **Dispatch the three lenses in one message per round** — that's what makes them parallel and independent.
2. **Round 1 is isolated** — no ledger, no cross-talk; let each lens form a genuine first read.
3. **The Arbiter adjudicates, never a subagent** — scoring and elimination are yours, applied by the fixed bracket rules with no randomness.
4. **Anti-fabrication is sacred** — a lens that invents points to seem rigorous corrupts the gauge. Reward honest "nothing material here"; score speculation to 0 and let it die.
5. **`--emphasis` tunes tone, never scores** — the user cannot dial in their preferred verdict.
6. **Converge, don't pad** — stop when the surviving set stabilizes for two rounds; a settled debate gains nothing from more rounds.
7. **Flag borrowed confidence** — any `(assumed)` framing or loaded-premise input rides along as a standing blind spot and is named again at verdict time.
8. **Model selection:** default the lens subagents to `model: "sonnet"` — you may run 9–30 of them and the charters carry the rigor. Use `model: "opus"` for genuinely high-stakes calls (architecture, compliance, irreversible spend). The Arbiter's adjudication and synthesis is where depth matters most.
9. **Display progressively** — show each round's contentions and the running ledger as you go, not just the final matrix.
10. **Turbo/lite fallback** — if subagents are unavailable, the idea is trivial, or the user asks for "quick", simulate the triad in one context and say so in the mode line. Never silently downgrade.
11. **A flat or balanced result is a valid result** — don't manufacture drama on a solid idea or a coin-flip; report it straight.
12. **Corroboration is signal, never noise** — never merge a contention across lenses to tidy the ledger; independent agreement raises Survivability and shows as multi-lens weight in the gauge. Merging is only for one lens repeating itself.
