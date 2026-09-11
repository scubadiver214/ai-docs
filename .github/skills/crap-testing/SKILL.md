---
name: crap-testing
description: "Compute and act on CRAP (Change Risk Anti-Patterns) scores — Savoia & Evans's metric that combines cyclomatic complexity with test coverage to flag methods that are both complex and undertested. Use this skill when the user asks about CRAP score, change-risk analysis, where to focus testing effort, which methods are 'crappy', how to prioritize refactor-vs-test decisions, or how to wire crap4j / JaCoCo CRAP reporting into a build. Also use when the user wants a triage pass over a codebase to rank methods by change risk rather than by raw coverage or raw complexity."
---

# CRAP Testing — Change Risk Anti-Patterns

The CRAP score answers one question: **which methods are most likely to break when changed, and what's the cheapest fix?** A method earns a high CRAP score when it is _both_ complex _and_ poorly tested. Either condition alone is tolerable; the combination is where bugs live.

## Formula

```
CRAP(m) = comp(m)² × (1 − cov(m)/100)³ + comp(m)
```

- `comp(m)` — cyclomatic complexity of method `m` (count of linearly independent paths; +1 per `if`, `else if`, `case`, `&&`, `||`, `?:`, `catch`, loop).
- `cov(m)` — test coverage of `m` as a percentage. Prefer **branch coverage** over line coverage; line coverage hides untested conditionals and inflates the score in your favor.

### Interpreting the score

| CRAP  | Verdict    | Action                                 |
| ----- | ---------- | -------------------------------------- |
| ≤ 5   | Clean      | Leave it alone.                        |
| 6–15  | Acceptable | Monitor; fix opportunistically.        |
| 16–30 | Risky      | Schedule work — add tests or simplify. |
| > 30  | **Crappy** | Block further changes until addressed. |

The threshold of 30 is the canonical Savoia/Evans cutoff. Adjust per project, but state the threshold explicitly when reporting.

### Why the exponents matter

The `²` on complexity and `³` on the uncovered fraction are deliberate: CRAP punishes the _combination_ much harder than either factor alone. A method with complexity 10 and 100% coverage scores 10. The same method with 0% coverage scores 110. Drop coverage from 100% to 80%, and the score barely moves (10.8) — which is the point. Tests buy down risk non-linearly.

## How to use this skill

### 1. Computing CRAP for a single method

When the user pastes a method or points at one, do this:

1. Count cyclomatic complexity by hand: start at 1, add 1 for each branching construct (`if`, `else if`, `case`, `&&`, `||`, ternary, `catch`, `for`, `while`, `?.` in some languages). Don't count `else` or `default` — they don't add a path.
2. Ask for or estimate branch coverage. If only line coverage is available, say so and treat the result as an upper bound on safety (i.e., a lower bound on real CRAP).
3. Plug into the formula. Show the substitution, not just the answer.
4. State the verdict and the cheaper of the two remediations (see §3).

**Worked example.** A method with 6 branches and 40% branch coverage:

```
CRAP = 6² × (1 − 0.40)³ + 6
     = 36 × 0.216 + 6
     = 7.776 + 6
     = 13.78  →  Acceptable, but trending risky.
```

### 2. Computing CRAP across a codebase

Don't compute by hand at scale. Use the language-appropriate tool:

| Stack                   | Tool                                                               | Notes                                                                                                                  |
| ----------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| Java                    | **crap4j** (legacy) or JaCoCo + custom report                      | crap4j hasn't been updated in years; most teams now derive CRAP from JaCoCo XML + a complexity tool (PMD, Checkstyle). |
| .NET                    | **NCrunch**, **NDepend**                                           | NDepend exposes CRAP directly in its dashboards.                                                                       |
| JavaScript / TypeScript | `escomplex` / `complexity-report` + Istanbul (`nyc`) coverage JSON | Join on function name + file.                                                                                          |
| Python                  | `radon cc` + `coverage.py` JSON                                    | `radon` gives complexity; `coverage json` gives per-line hits — aggregate to function level.                           |
| Ruby                    | `flog` + SimpleCov                                                 | Approximate; flog's score is not strictly cyclomatic.                                                                  |
| Go                      | `gocyclo` + `go test -coverprofile`                                | Parse coverprofile per-function.                                                                                       |

When asked to set this up in CI:

- Emit the CRAP report as a build artifact (CSV or JSON), not just a console log. Trends matter more than snapshots.
- **Fail the build on regression, not on absolute score.** A legacy codebase will have many crappy methods on day one. Block PRs that _introduce_ new methods with CRAP > 30 or that push an existing method's score higher.
- Sort the report by CRAP descending and surface the top 20 in PR comments.

### 3. Triage: test it or refactor it?

Every crappy method has two exits. The formula tells you which is cheaper.

- **High complexity, decent coverage** (e.g., comp 15, cov 70%) → **refactor first**. Adding tests to a 15-branch method means writing many tests and they will be fragile. Extract methods until each piece has comp ≤ 5, _then_ test.
- **Modest complexity, low coverage** (e.g., comp 6, cov 20%) → **test first**. The method isn't the problem; the absent tests are. A handful of tests will collapse the score.
- **Both bad** (e.g., comp 20, cov 10%) → refactor under the safety net of _characterization tests_. Write tests that pin current behavior (even if the behavior is wrong), refactor, then fix behavior with the tests as a ratchet. Don't try to do all three at once.

State the recommendation in those terms: "this is a refactor problem" or "this is a coverage problem", not just "CRAP is high."

### 4. What CRAP does _not_ measure

Be explicit about the metric's blind spots when reporting — don't oversell it:

- **Coupling.** A simple, well-tested method that calls into a tangle is still risky. CRAP misses this; pair it with a coupling metric (afferent/efferent, or a dependency graph) for a fuller picture.
- **Test quality.** Coverage records that a line ran, not that the assertion was meaningful. A method with 100% coverage and no assertions scores like a fully-tested one. If the user is gaming coverage, CRAP gives them cover.
- **Mutation survival.** The honest version of "is this tested" is mutation testing (Pitest, Stryker, mutmut). When stakes are high, recommend mutation score over coverage as the `cov(m)` input.
- **Change frequency.** A crappy method nobody touches is a lower priority than a moderately-crappy method edited every sprint. Cross-reference with `git log --follow` churn before declaring a remediation order.

## Output format when reporting

When you produce a CRAP analysis, structure it like this:

1. **Top offenders** — table of method, file:line, complexity, coverage, CRAP, verdict. Cap at 20 rows.
2. **Recommendation per offender** — one line each: "test", "refactor", or "characterize-then-refactor", with a one-clause reason.
3. **Codebase summary** — count of methods in each verdict bucket, and the trend vs. the previous report if available.
4. **Caveats** — which blind spots from §4 apply to this codebase (e.g., "coverage is line-based; treat scores as optimistic").

Keep the prose short. The table is the deliverable.
