## Examples: Simple → Complex

### 1. Simplest — Quick Discussion (one line)

```
/council discuss whether we should use DynamoDB or PostgreSQL for user session storage
```

**What happens:** 3 auto-selected agents (likely `aws-database-architect-master`, `software-engineering-master`, `hipaa-compliance-master`), 2 rounds of Socratic exploration, synthesis with recommendations.

---

### 2. Targeted Review — Named Agent Count

```
/council 4 experts review our current JWT validation middleware
```

**What happens:** 4 agents auto-selected for security/auth relevance (e.g., `cognito-jwt-auditor`, `principal-sdet-master`, `software-engineering-master`, `hipaa-phi-boundary`). Each grades the code A-F, identifies top 3 risks, 2 rounds, synthesized recommendations.

---

### 3. Debate with Named Skills

```
/council debate with prompt-engineering-master and software-engineering-master whether RAG or fine-tuning is better for Benefitly's document intelligence
```

**What happens:** 2 named experts take opposing positions. 3 rounds: thesis → antithesis → synthesis. The moderator identifies where they converged and where they didn't.

---

### 4. Artifact Improvement — File Path

```
/council improve /mnt/skills/user/auto-deliberate/SKILL.md — bring 5 experts including prompt-engineering-master and principal-sdet-master
```

**What happens:** Reads the file first. 5 agents (2 named + 3 auto-inferred) run 3 rounds: propose improvements → critique each other's suggestions → produce refined version. Synthesis includes the final improved artifact.

---

### 5. Complex — Full Specification

```
/council 6 experts debate whether Benefitly should migrate from Cognito to Auth0, considering HIPAA compliance, developer experience, cost at scale, and multi-tenant isolation. Include hipaa-compliance-master, cognito-user-lifecycle, aws-database-architect-master, and software-engineering-master.
```

**What happens:** 6 agents (4 named + 2 auto-inferred to cover cost/DX angles). 3 adversarial rounds. Each expert argues from their domain — the HIPAA expert cares about BAAs, the Cognito expert defends the incumbent, the architect evaluates migration risk, the cost analyst models pricing at 10k/100k/1M users. Synthesis captures the full decision matrix.

---

### 6. Most Complex — Multi-Artifact Review Pipeline

```
/council review the entire Benefitly compliance posture — examine our BAA template, our encryption patterns, our audit logging pipeline, and our PHI access controls. Bring hipaa-compliance-master, encryption-pattern-validator, audit-pipeline-hardening, soc2-compliance-master, and principal-sdet-master.
```

**What happens:** 5 specialists each review multiple artifacts through their lens. 2 rounds of rigorous evaluation with letter grades and risk rankings. Synthesis produces a prioritized remediation roadmap across all compliance domains.

---

## Flexibility Summary

| Dimension         | Range                                                    |
| ----------------- | -------------------------------------------------------- |
| **Agents**        | 2–6, named or auto-inferred                              |
| **Modes**         | debate, discuss, improve, review                         |
| **Rounds**        | 2–3, mode-dependent                                      |
| **Input**         | Pure question, code, file path, or pasted artifact       |
| **Skill catalog** | 30+ specialized roles, mix-and-matched per topic         |
| **Invocation**    | Single natural-language line — no config files, no flags |
