# ADR-003: Skill vs. Tool Boundary

**Status**: Accepted
**Date**: 2026-06-23
**Deciders**: Solution Architect (Morgan)
**Feature**: sre-agent-mttr-optimizer

---

## Context

Claude Code provides two distinct primitives for giving an agent access to external knowledge and capabilities:

1. **Skills** (`.claude/skills/{name}/SKILL.md`) — Markdown instruction files injected into agent context at runtime via the `Skill` tool. They encode domain knowledge, reasoning procedures, classification frameworks, and decision heuristics. Skills are pure context — they do not execute code, make API calls, or produce side effects.

2. **Tools** — Bash commands, API calls, file reads, and agent invocations available to an agent during execution. Tools produce side effects: they read from the filesystem, call external APIs, or invoke sub-agents. In the Mastra model, tools are `createTool({ execute: async () => {} })` functions.

For the SRE MTTR-optimizer, this boundary must be explicit because:
- SRE domain knowledge (how to interpret an alert, what confidence thresholds mean, how to cite evidence) belongs in Skills
- Data retrieval (fetching OTel traces, reading log lines, querying Prometheus) belongs in Tools
- Mixing them creates untestable, non-composable agent definitions

The constraint C-04 (output must be explainable at the hypothesis level) and C-03 (runbook steps validated against current state) both depend on this boundary being clean: Skills define the reasoning rules, Tools provide the data the rules apply to.

---

## Decision

**Skills encode reasoning procedures and domain heuristics. Tools execute and produce data. An agent's intelligence is in its Skills; its reach is in its Tools.**

### What goes in a Skill

- **Alert triage heuristics**: how to classify an alert as causal vs. symptomatic vs. noise; threshold rules for alert storm severity; deduplication logic for correlated alert groups
- **Signal correlation procedures**: how to join a trace ID from a log line to an OTel span; how to correlate a metric spike timestamp to a trace start time; how to infer causality direction from service dependency graphs
- **Hypothesis generation rules**: confidence score calculation criteria; how to rank competing hypotheses; citation format for evidence ("Hypothesis H1 is supported by: trace 7f3a showing 450ms P99 latency spike at 03:14:22 UTC, correlated with alert PROD-4421 firing at 03:14:19 UTC")
- **Runbook validation procedures**: how to check whether a runbook step references a service that still exists; how to surface "step may be stale" vs. "step is definitely invalid" vs. "step validated against current state"
- **Graceful degradation rules**: what to output when OTel traces are absent ("signal gap detected: trace coverage unavailable for service X — proceeding with log+metrics correlation only")
- **Trust and autonomy classification**: which action categories are read-only (full autonomy), which require confirmation, which must never be autonomous (C-01 constraint)

### What goes in a Tool

- **OTel trace fetcher**: calls the OTel collector or backend API; returns raw trace spans for a time window and service scope
- **Metrics fetcher**: calls Prometheus / metrics backend; returns time-series data points for alert-relevant metric names
- **Log fetcher**: calls log aggregation backend or reads log files; returns structured log entries for a time window and service scope
- **Alert list fetcher**: calls PagerDuty API; returns active alerts with severity, timestamps, and linked services
- **Runbook reader**: reads Markdown runbook files from the local filesystem (C-05 constraint: Git-resident Markdown primary format); returns structured runbook content
- **Audit logger**: writes structured audit entries to a log file or event stream (C-01 constraint: action audit trail)
- **Slack notifier**: posts formatted incident brief to a configured Slack channel (output surface)
- **System state prober**: executes read-only system state queries (e.g., `kubectl get pods`, `curl health endpoint`) to validate runbook step applicability (C-03 constraint)

### Boundary enforcement rule

> If the operation produces output by reasoning over data, it is a Skill.
> If the operation produces data by interacting with the outside world, it is a Tool.

A "tool that also interprets its results" is a violation of this boundary. The tool returns raw data; the agent (guided by its Skill) interprets it.

---

## Alternatives Considered

### Alternative 1: Skills encode everything including data schemas and tool call patterns

**Description:** Skills include embedded examples of what tool outputs look like and reasoning patterns applied to those examples. This is a "fat skill" approach.

**Pros:**
- Agent context is self-contained — no need to separately maintain tool documentation

**Cons:**
- Skills become coupled to tool output schemas; changing a tool's output format requires updating the Skill
- Skills that contain data examples inflate context window size with token-expensive examples
- Testing a Skill's reasoning procedure in isolation becomes impossible because it references specific tool output shapes

**Rejection rationale:** Violates the principle of Effect Isolation by Design (Principle 12). Skills should be pure context (Functional Core); tools are the Imperative Shell. Coupling them creates an untestable blob.

### Alternative 2: No Skills — all domain knowledge in agent system prompts

**Description:** Each agent's system prompt contains both the domain knowledge and the procedural instructions inline. No separate Skill files.

**Pros:**
- Simpler file structure; no need to manage skill injection
- Agent system prompt is the single source of truth for agent behaviour

**Cons:**
- Domain knowledge is duplicated across agents that share heuristics (e.g., alert classification rules needed by both the triage agent and the hypothesis ranker)
- Updating a shared heuristic (e.g., changing the confidence threshold from 0.7 to 0.8) requires editing every agent system prompt — violates DRY and creates consistency risk
- Mastra porting is harder: domain knowledge embedded in system prompts cannot be extracted as Mastra context/memory without rework

**Rejection rationale:** Skills are the reuse mechanism. Without them, shared domain knowledge becomes duplicated and drift-prone across agent boundaries.

### Alternative 3: Tools include LLM-based interpretation (smart tools)

**Description:** Tools call sub-LLMs internally to interpret their outputs before returning them. For example, the log fetcher calls an LLM to pre-classify log entries before returning them to the parent agent.

**Pros:**
- Parent agent receives pre-processed, interpreted data — simpler agent reasoning

**Cons:**
- Hidden LLM calls inside tools are opaque to the Workflow orchestrator; cost, latency, and failure modes are invisible
- Violates Earned Trust (Principle 13): a sub-LLM is an external dependency with no probe contract
- Creates nested LLM call chains that are extremely difficult to debug during the walking skeleton stage
- Cost is unpredictable: tool invocations that "also reason" make token usage non-deterministic

**Rejection rationale:** The walking skeleton stage requires maximum debuggability. Smart tools hide complexity. Tools must be transparent data producers.

---

## Consequences

### Positive
- Skills are reusable across agents: the alert triage heuristics Skill can be injected into both `alert-triage-agent` and `hypothesis-ranker-agent` without duplication
- Tools are unit-testable without LLM involvement: a Bash tool that calls the OTel API can be tested with a mock OTel server
- The Skill-vs-Tool boundary is the same boundary that Mastra exploits: Skills port as agent `instructions` + context injection; Tools port as Mastra `createTool()` definitions
- Effect isolation is structurally enforced: Skills have no `execute` function, no side effects, and cannot mutate state — violations are syntactically impossible in the Markdown format

### Negative
- Requires discipline to maintain the boundary during development; over time, procedural knowledge may migrate into tool implementations ("just add a quick interpretation step to the tool")
- Skill files must be updated when data schemas change if the Skill references field names — requires a naming stability contract with Tool authors

### Enforcement

- CI pre-commit hook: Skill files (`*.md` in `.claude/skills/`) must not contain code blocks that execute shell commands or make HTTP calls — enforced by a regex scan
- Code review checklist: every new Tool implementation must be reviewed against the question "does this Tool interpret its results or just return them?" — any interpretation logic must be moved to a Skill
- Mastra port review: when porting Tools to Mastra `createTool()`, review that the `execute` function returns raw data only
