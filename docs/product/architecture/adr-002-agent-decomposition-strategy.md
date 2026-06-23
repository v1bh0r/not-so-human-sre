# ADR-002: Agent Decomposition Strategy

**Status**: Proposed — Morgan recommends Option B (Three-Stage Pipeline); awaiting user confirmation before Accepted
**Date**: 2026-06-23
**Deciders**: Solution Architect (Morgan), Product Owner
**Feature**: sre-agent-mttr-optimizer

> **Note for reviewers**: This ADR is intentionally Proposed, not Accepted. The architecture brief presents all three options and recommends Option B, but the user has not yet confirmed their selection. The architecture brief's "Option B" sections describe the recommended design, not the decided design. Once the user selects an option, this ADR will be updated to Accepted with the confirmed decision rationale, and the other options' sections will be marked as "rejected".

---

## Context

The SRE MTTR-optimizer must address the ODI cluster (ODI-01, ODI-03, ODI-04) that all trace to the same root: the SRE is the manual correlation engine across multiple signal types and tools. The agent system must automate that pipeline.

The pipeline has three semantically distinct phases:
1. **Alert intake + noise reduction** — reduce 300 firing alerts to the causal subset (addresses ODI-04)
2. **Cross-signal correlation** — join logs, metrics, and traces into a single causal chain (addresses ODI-03)
3. **Hypothesis generation** — rank candidate root causes with confidence scores and evidence citations (addresses ODI-01)

The key design decision is how to decompose this pipeline into Claude Code Agents and how the Workflow orchestrator coordinates them.

Three decomposition strategies were analysed. This ADR documents them and their trade-offs. **The chosen strategy is populated here once the user selects an option** (see `docs/feature/sre-agent-mttr-optimizer/design/wave-decisions.md`).

---

## Option A: Single Triage Agent (Monolithic)

**Summary:** One agent handles alert intake, signal correlation, and hypothesis generation in sequence. The Workflow script calls this one agent and extracts structured output.

**Agent topology:**
- `sre-triage-agent` — receives the alert context, calls tools for OTel traces, metrics, logs, alert list; generates the incident brief as structured JSON

**Workflow structure:**
```
Workflow: incident-triage
  └── Step 1: sre-triage-agent(alert_context) → IncidentBrief
```

**Pros:**
- Simplest implementation — one agent system prompt to write and debug
- Lowest latency (no inter-agent handoffs)
- Context is never fragmented across agent boundaries; the agent holds the full picture throughout
- Easiest to port to Mastra (single agent definition)

**Cons:**
- Agent system prompt grows large and becomes a "god prompt" — hard to isolate which instruction is causing a failure
- Alert noise reduction and hypothesis generation are semantically distinct tasks; combining them in one agent risks conflation (the agent may prematurely correlate noisy alerts into hypotheses)
- Adding a new signal type (e.g., deployment events, change tracking) requires modifying the single agent's instructions and tool list — no isolation
- Context window pressure: a large alert storm + log batch + trace bundle may approach the limit before the agent has reasoned through all phases
- Skill injection is coarse-grained: one large skill file for the entire SRE domain, or multiple skills all injected into one agent

**Best for:** Earliest-possible prototype. Validated correctness of OTel signal parsing before adding agent coordination complexity.

---

## Option B: Three-Stage Pipeline (Recommended)

**Summary:** The pipeline is decomposed into three focused agents, each responsible for one semantic phase. The Workflow orchestrates them in sequence, passing structured output between stages. Alert correlation within a stage runs in parallel where signal types are independent.

**Agent topology:**
- `alert-triage-agent` — reduces alert storm to causal subset; outputs `AlertSubset { causalAlerts[], noiseAlerts[], triageRationale }`
- `signal-correlator-agent` — receives causal alerts + fetches OTel traces, metrics, logs in parallel; outputs `CorrelatedSignals { causalChain[], signalGaps[], timelineEvents[] }`
- `hypothesis-ranker-agent` — receives correlated signals; outputs `IncidentBrief { hypotheses[{rank, confidence, evidence, explanation}], recommendedAction, auditTrail }`

**Workflow structure:**
```
Workflow: incident-triage
  ├── Step 1: alert-triage-agent(incident_context) → AlertSubset
  ├── Step 2 [parallel fan-out]:
  │     ├── fetch-otel-traces(causalAlerts) → TraceBundle
  │     ├── fetch-metrics(causalAlerts)     → MetricBundle
  │     └── fetch-logs(causalAlerts)        → LogBundle
  ├── Step 3: signal-correlator-agent(AlertSubset, TraceBundle, MetricBundle, LogBundle) → CorrelatedSignals
  └── Step 4: hypothesis-ranker-agent(CorrelatedSignals) → IncidentBrief
```

**Pros:**
- Each agent has a single, testable responsibility — failure isolation is precise
- Parallel signal fetching in Step 2 (OTel traces + metrics + logs) runs concurrently, reducing wall-clock time vs. serial fetching
- Skills map cleanly: `sre-alert-triage.md` skill for Step 1, `sre-signal-correlation.md` for Step 3, `sre-hypothesis-generation.md` for Step 4
- Context window is bounded per agent: triage agent never sees the full log bundle; correlator never generates hypotheses; ranker never re-reads raw alerts
- Adding a new signal type (e.g., deployment events) means adding a new parallel branch in Step 2 and updating the correlator's skill — no other agents change
- Most direct mapping to Mastra: each agent becomes a `new Agent()`, workflow steps map to Mastra `createStep()` instances
- Graceful degradation (C-02): if OTel traces are missing, `TraceBundle` contains `signalGap: true`; correlator handles this explicitly rather than silently

**Cons:**
- Three agent system prompts to write, tune, and test — higher upfront effort than Option A
- Intermediate structured outputs must be well-specified (JSON Schema required for each handoff) — more schema design work
- If hypothesis-ranker-agent needs to re-examine raw alerts, it must receive them from the workflow context — inter-agent data passing requires careful schema design
- Slightly higher latency than Option A due to serialized agent invocations (partially offset by parallel signal fetching in Step 2)

**Best for:** Walking skeleton that validates the full ODI cluster (ODI-01, ODI-03, ODI-04) with isolated, debuggable agents and clean Mastra porting path.

**This is the recommended option.** See rationale in Decisions Table.

---

## Option C: Parallel Specialist Swarm

**Summary:** All three signal types are processed by dedicated specialist agents simultaneously. A synthesis agent receives all three specialist outputs and produces the incident brief. This is the highest-parallelism topology.

**Agent topology:**
- `log-analyst-agent` — fetches and reasons over log streams; outputs `LogAnalysis`
- `metrics-analyst-agent` — fetches and reasons over metric time series; outputs `MetricsAnalysis`
- `trace-analyst-agent` — fetches and reasons over OTel traces; outputs `TraceAnalysis`
- `alert-filter-agent` — independently reduces alert noise; outputs `AlertSubset`
- `incident-synthesizer-agent` — receives all four specialist outputs; generates `IncidentBrief`

**Workflow structure:**
```
Workflow: incident-triage
  ├── Step 1 [full parallel fan-out]:
  │     ├── log-analyst-agent(incident_context)     → LogAnalysis
  │     ├── metrics-analyst-agent(incident_context) → MetricsAnalysis
  │     ├── trace-analyst-agent(incident_context)   → TraceAnalysis
  │     └── alert-filter-agent(incident_context)    → AlertSubset
  └── Step 2: incident-synthesizer-agent(all four outputs) → IncidentBrief
```

**Pros:**
- Maximum parallelism: all signal analysis runs simultaneously; theoretical lowest wall-clock time
- Specialists can be independently swapped or upgraded (e.g., replace `log-analyst-agent` with a model fine-tuned on log patterns)
- Synthesizer has complete, pre-analyzed views of all signals; does not need to reason about raw data
- Fault isolation per signal type: if trace analysis fails, log and metrics analysis still complete

**Cons:**
- Five agent prompts to write and coordinate — significantly higher upfront complexity for a walking skeleton
- The synthesizer must reconcile four independent analyses that may use inconsistent terminology or causal frames — "correlation" happens in language at synthesis time, which is harder to verify than structured data joins in Option B
- Alert reduction (Step 1 `alert-filter-agent`) runs in parallel with signal analysis, but signal analysts receive the full alert context — they may analyze signals for alerts that the filter would have dropped, wasting tokens
- If specialists disagree (e.g., log analyst says service A is the cause, metrics analyst says service B), the synthesizer has no ground truth to reconcile them — it must resolve ambiguity through language, reducing determinism
- Highest LLM cost per incident: five concurrent agents × context size
- Most complex Mastra port: five agents + parallel workflow step fan-out must be precisely coordinated

**Best for:** Production-scale system where each signal type has enough volume and complexity to justify a dedicated specialist. Not appropriate for walking skeleton stage.

---

## Chosen Strategy

[To be populated after user selects option. See `docs/feature/sre-agent-mttr-optimizer/design/wave-decisions.md`.]

**If Option B (recommended):** The three-stage pipeline with parallel signal fetching in Stage 2.

---

## Enforcement

- Every inter-agent handoff MUST have a JSON Schema defined in the Workflow script
- No agent may produce free-form text as its primary output where a downstream agent or the Workflow depends on structured fields
- The Workflow script MUST validate output schema compliance after each agent invocation and emit a `workflow.step.schema_violation` event if violated — this is the structural probe for inter-agent contracts
- `import-linter` is not applicable to JS workflow scripts; schema validation in the Workflow runner is the enforcement mechanism
