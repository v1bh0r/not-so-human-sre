# Job Analysis — sre-agent-mttr-optimizer

Wave: DIVERGE
Phase: 1 — JTBD Analysis
Date: 2026-06-23
Skill: nw-jtbd-analysis

---

## Raw Request

Problem statement (from DISCOVER wave, validated across 10 interviews):

"SREs lose 10–30 minutes per incident manually correlating logs, metrics, and traces before forming their first diagnostic hypothesis. This is the primary MTTR driver. 100% of 10 synthesised interviews confirmed this pain."

Top validated opportunities:
- OPP-01: Automated cross-signal correlation → "incident brief" in <60s (score: 15.5)
- OPP-02: Alert noise reduction to causal subset (score: 14.5)
- OPP-03: Ranked hypothesis generation with confidence + evidence (score: 14.0)

---

## Job Extraction — 5 Why Chain

Starting point (tactical): "Correlate logs, metrics, and traces during an incident."

**Why do SREs need to correlate logs, metrics, and traces?**
To form a diagnostic hypothesis — to understand what is failing and why, before acting.

**Why do they need a hypothesis before acting?**
Because acting without a verified hypothesis risks the wrong mitigation: either wasting time on the wrong service, or — worse — destabilizing a partially functional system.

**Why is forming the hypothesis manual today?**
Because no tool automatically connects the causal chain across the three signal types (logs, metrics, traces) in the context of a specific live incident. SREs are the correlation engine themselves.

**Why does it matter that the SRE is the correlation engine?**
Because the SRE's cognitive capacity is the bottleneck. Every minute spent on correlation is a minute not spent on mitigation. At P0 severity, that bottleneck has direct financial and reputational consequences.

**Why is minimizing that cognitive bottleneck the real goal?**
Because the actual job is not "correlate signals" — it is "restore service reliability with minimum downtime and minimum blast radius." Signal correlation is overhead on the way to that job. Nobody values the correlation itself; they value the confidence and speed it produces.

**Abstraction level reached: Strategic.**
Job: "Restore service reliability under incident conditions with minimum delay and minimum risk of compounding the failure."

Disruption check: Is there a higher-level job that would make this entire job unnecessary?
Yes — "prevent incidents from occurring." But prevention and response are separate markets. The SRE response job exists for incidents that prevention did not catch. This is not eliminable by going up another level; it is appropriately scoped.

---

## Job Statements

### Functional Job (primary)

When a production alert fires during an active incident, I want to immediately understand what is failing, where it is failing, and why it is failing, so I can begin targeted mitigation instead of investigation.

**Abstraction level**: Strategic (eliminates diagnostic overhead entirely, not just improves it).

**Validated by**: 8/10 DISCOVER interviewees described the pre-hypothesis investigation phase as their primary MTTR driver; 0/10 valued the correlation work itself — they valued what it produces.

### Emotional Job

When a production system is failing and I am the person paged, I want to feel immediately oriented and in command of the situation, so I can act with confidence rather than anxiety, and know that I am not missing something obvious that I will be blamed for later.

**Evidence**: I-06: "The worst part isn't the fix — it's the first fifteen minutes where you don't know what you don't know." I-05: "Give me the top 3 hypotheses and why, and I'll pick one." The desire is for felt competence under pressure, not raw information.

### Social Job

When an incident is resolved and reviewed, I want to be seen as the person who ran a clean, fast, well-documented response, so that my organization attributes MTTR improvement to my team's capability — not luck.

**Evidence**: I-02: "MTTR is tracked. Every minute of P1 costs money and career capital." I-10: "We track MTTR in a spreadsheet because our incident tool doesn't slice it the way we need."

---

## Disruption Check

**Could a higher-level job make this entire job unnecessary?**

The higher-level job is: "Maintain service reliability without requiring human incident response." This is the incident-prevention + auto-remediation direction. It is not a disruption threat to this product at MVP stage because:

1. Zero interviewees trusted AI for fully autonomous remediation on non-trivial incidents (INV-02 validated).
2. The "ghost in the chair" pattern — AI investigates, human decides — is the highest-resonance design metaphor (8/10).
3. The job of informed human decision-making under incident conditions is durable even as AI capabilities expand.

Disruption risk: LOW at 3-year horizon. Medium at 5+ years as trust in AI remediation grows.

---

## ODI Outcome Statements

Using format: `[Direction] + [Metric] + [Object] + [Context]`

**ODI-01**: Minimize the time it takes to form a first plausible diagnostic hypothesis after a production alert fires.

- Importance (from DISCOVER): 9.5/10 (tied-highest with cross-signal correlation)
- Current satisfaction: 3/10
- Opportunity score: 9.5 + (9.5 - 3) = **16.0** — severely under-served

**ODI-02**: Minimize the likelihood of acting on a mitigation strategy that is based on an incorrect or incomplete signal interpretation.

- Importance: 9/10 (consequence of wrong hypothesis is compounded failure)
- Current satisfaction: 4/10 (manual correlation is imprecise, especially at 3am under stress)
- Opportunity score: 9 + (9 - 4) = **14.0** — under-served

**ODI-03**: Minimize the number of monitoring tools and tabs an SRE must actively query to construct a complete picture of an incident.

- Importance: 8.5/10 (tool sprawl is the mechanism of the correlation tax)
- Current satisfaction: 2.5/10 (I-03: "I have three tabs open — Grafana, New Relic, and our runbook wiki. I'm the correlation engine.")
- Opportunity score: 8.5 + (8.5 - 2.5) = **14.5** — under-served

**ODI-04**: Minimize the likelihood of the on-call SRE missing a causal alert signal because it is buried in alert storm noise.

- Importance: 9/10 (I-06: "We have 4,000 alert rules. During a major incident we might see 300 fire simultaneously. Maybe 8 are causal.")
- Current satisfaction: 3.5/10
- Opportunity score: 9 + (9 - 3.5) = **14.5** — under-served

**ODI-05**: Minimize the effort required to communicate a coherent incident status to stakeholders and incident commanders without interrupting the active investigation.

- Importance: 8/10 (I-02: "I don't have time to read logs myself... I need a coherent narrative")
- Current satisfaction: 3/10 (today this is a separate cognitive task — pause investigation, write update, resume)
- Opportunity score: 8 + (8 - 3) = **13.0** — under-served

**ODI-06**: Minimize the time it takes to validate that a candidate runbook step is applicable to the current system state before executing it.

- Importance: 8.5/10 (wrong runbook step can cause secondary incidents)
- Current satisfaction: 3/10 (I-04: "I'd estimate 30% of our runbooks are actually current")
- Opportunity score: 8.5 + (8.5 - 3) = **14.0** — under-served

---

## Outcome Opportunity Summary

| ODI | Statement (condensed) | Importance | Satisfaction | Score | Status |
|-----|----------------------|------------|--------------|-------|--------|
| ODI-01 | Time to first diagnostic hypothesis | 9.5 | 3.0 | 16.0 | Severely under-served |
| ODI-02 | Likelihood of acting on wrong interpretation | 9.0 | 4.0 | 14.0 | Under-served |
| ODI-03 | Tools/tabs queried to build incident picture | 8.5 | 2.5 | 14.5 | Under-served |
| ODI-04 | Likelihood of missing causal alert in noise | 9.0 | 3.5 | 14.5 | Under-served |
| ODI-05 | Effort to communicate status without interrupting investigation | 8.0 | 3.0 | 13.0 | Under-served |
| ODI-06 | Time to validate runbook step against current state | 8.5 | 3.0 | 14.0 | Under-served |

**Most under-served**: ODI-01 (score 16.0). Every design direction must demonstrate a mechanism for reducing time-to-first-hypothesis.

**Cluster**: ODI-01, ODI-03, and ODI-04 are structurally linked — all three are aspects of the same underlying problem (the SRE is the correlation engine, working across too many signals, with too much noise). A solution that addresses the cluster (not just one) will have the highest opportunity score capture.

---

## Gate G1 Validation

| Criterion | Status | Evidence |
|-----------|--------|----------|
| Job at strategic or physical level | PASS | "Restore service reliability with minimum delay and minimum risk" — strategic, not tactical |
| No feature references in job statement | PASS | "Correlate logs" is not in the functional job statement — it appears only in the 5-Why extraction as a tactical starting point |
| Minimum 3 ODI outcome statements | PASS | 6 ODI outcome statements produced |
| All ODI statements follow correct format | PASS | Direction + Metric + Object + Context format used throughout |

**G1: PASS. Proceed to Phase 2 (Competitive Research) and Phase 3 (Brainstorming).**
