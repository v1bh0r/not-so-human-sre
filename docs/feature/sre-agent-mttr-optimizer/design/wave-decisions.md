# DESIGN Wave Decisions — sre-agent-mttr-optimizer

**Wave**: DESIGN
**Date**: 2026-06-23
**Architect**: Morgan (nw-solution-architect)
**Mode**: PROPOSE

---

## Decision 1: Runtime

**Question**: Which runtime for the walking skeleton?

**Options**:
- Claude Code Agents + Skills + dynamic Workflows
- Mastra.ai directly
- LangChain.js
- Plain Anthropic SDK

**Chosen**: Claude Code Agents + Skills + dynamic Workflows (ADR-001, Accepted)

**Rationale**: User explicitly requested Claude Code primitives for the walking skeleton. Mastra remains the declared production target. Decouple agent design validation from framework coupling risk (A-10: MEDIUM confidence on Mastra feasibility).

**Status**: Accepted — not subject to user selection

---

## Decision 2: Agent Decomposition Strategy

**Question**: How is the triage/correlation/hypothesis pipeline decomposed into Claude Code Agents?

**Options presented**:

| Option | Name | Agent Count | Recommended For |
|--------|------|-------------|-----------------|
| A | Single Triage Agent | 1 | PoC only |
| B | Three-Stage Pipeline | 3 | Walking skeleton (RECOMMENDED) |
| C | Parallel Specialist Swarm | 5 | Mature production system |

**Morgan's recommendation**: Option B

**Key differentiator**: Option B is the only option that structurally enforces alert noise reduction (Stage 1) as a typed gate before cross-signal correlation begins (Stage 3). This is the mechanism for ODI-04. Options A and C cannot enforce this structurally — it is a convention in A and a timing gap in C (alert-filter-agent and signal analysts run in parallel, so analysts may process noise-correlated signals before filtering completes).

**Status**: PENDING USER SELECTION

---

## Decision 3: Skill vs Tool Boundary

**Question**: What domain knowledge goes in a Skill file vs. a Tool implementation?

**Rule chosen**: Skills encode reasoning procedures and domain heuristics. Tools execute and produce data. (ADR-003, Accepted)

**Status**: Accepted — not subject to user selection

---

## Decision 4: Structured Output Contract Between Agents

**Question**: How are inter-agent handoffs typed and validated?

**Chosen**: Zod v4 runtime JSON Schema validation per inter-agent handoff, enforced by the Workflow script after each agent invocation.

**Rationale**: Zod v4 is already a declared project dependency. Runtime validation provides immediate failure detection when an agent returns non-conforming output, which is critical during the walking skeleton iteration cycle. TypeScript types alone cannot catch runtime LLM output deviations.

**Status**: Accepted — not subject to user selection

---

## Decision 5: Adapter Probe Pattern

**Question**: How do adapters demonstrate they can honor their contracts in the real environment?

**Chosen**: Wire → Probe → Use composition root invariant. Every adapter implements `probe(): Promise<ProbeResult>`. All probes run before the first real alert is accepted. Probe failure emits `health.startup.refused` and halts.

**Enforcement**: Three-layer (subtype check, AST pre-commit, CI behavioral test).

**Status**: Accepted — not subject to user selection

---

## User Selection Required

**DECISION 2 is open.** Please select your preferred architecture option:

**Option A — Single Triage Agent**
One agent handles all three pipeline phases in sequence. Fastest to build, lowest debuggability. Best for a 3–5 day proof-of-concept spike before committing to Option B.

**Option B — Three-Stage Pipeline (Recommended)**
Three focused agents with parallel signal fetching between Stage 1 and Stage 3. Moderate build time (2–3 weeks). Highest debuggability. Best ODI structural coverage. Cleanest Mastra port.

**Option C — Parallel Specialist Swarm**
Five specialist agents with a synthesis agent. Highest parallelism, highest LLM cost, highest upfront build complexity. Appropriate for production v2 after the walking skeleton is validated.

---

## Post-Selection Actions

Once the user selects an option:

1. Update ADR-002 status from Proposed to Accepted with the chosen option
2. Create the `.claude/agents/` directory with agent definition files for the chosen option
3. Create the `.claude/skills/` directory with skill files for the chosen option
4. Create the Workflow orchestration script for the chosen option
5. Implement adapter stubs with `probe()` method signatures
6. Hand off to acceptance-designer (nw-acceptance-designer) for walking skeleton acceptance tests

---

## Constraints Traceability

| Constraint | Addressed By |
|-----------|--------------|
| C-01: Autonomous execution opt-in + audit log | `AuditLogPort` (bounded-change, append-only); no write tools in walking skeleton |
| C-02: Graceful degradation on incomplete OTel | `signalGap: boolean` typed field in Stage 2 output; correlator Skill handles gap explicitly |
| C-03: Runbook steps validated against system state | `hypothesis-ranker-agent` + `GitMarkdownAdapter` + runbook validation Skill (Option B Stage 4) |
| C-04: Hypothesis explainability | `evidence: string[]` required field in IncidentBrief schema; Skill defines citation format |
| C-05: Markdown runbook primary format | `GitMarkdownAdapter` with `marked` parser; primary format enforced at adapter level |
| C-06: Primary persona is on-call SRE, not post-mortem | Walking skeleton scope is live-incident only; post-mortem generation is out of scope |
