# ADR-001: Runtime — Claude Code Agents + Skills + dynamic Workflows vs Mastra.ai

**Status**: Accepted
**Date**: 2026-06-23
**Deciders**: Solution Architect (Morgan), Product Owner
**Feature**: sre-agent-mttr-optimizer

---

## Context

The `not-so-human-sre` project was bootstrapped with Mastra.ai (`@mastra/core` ^1.45.0). The team now wants to prototype the walking skeleton of the SRE MTTR-optimizer agent system using **Claude Code primitives** — Claude Code Agents (`.claude/agents/`), Skills (`.claude/skills/`), and the Claude Agent SDK dynamic Workflow tool — before committing to Mastra for production.

The business driver for this decision is risk management: Mastra.ai is a relatively new framework (A-10 from DISCOVER wave: "feasibility requires spike"). The team wants to validate agent design, prompt quality, and ODI coverage (ODI-01, ODI-03, ODI-04) on a stack they already have operational access to (Claude Code CLI), before incurring the full coupling cost of a Mastra production architecture.

**Constraints from DISCOVER wave:**
- C-01: Autonomous execution must be opt-in, reversible, and audit-logged
- C-04: Agent output must be explainable at the hypothesis level
- A-10 (MEDIUM confidence): Mastra as a production foundation requires feasibility spike

---

## Decision

Use **Claude Code Agents + Skills + Claude Agent SDK dynamic Workflows** as the walking skeleton runtime.

Mastra.ai remains the declared target production runtime. The walking skeleton will be architecturally isomorphic to the eventual Mastra design — agents map to Mastra agents, Skills map to Mastra agent instructions + context, Workflows map to Mastra workflows — enabling a porting path without rearchitecting.

---

## Alternatives Considered

### Alternative 1: Build directly on Mastra.ai from day one

**Pros:**
- Single implementation, no porting effort
- Mastra Studio provides built-in agent tracing (relevant to C-04 explainability)
- `@mastra/observability` provides OTel-native span tracking out of the box

**Cons:**
- A-10 is only MEDIUM confidence — framework behaviour under production load and complex multi-agent orchestration is unvalidated
- Mastra's workflow DSL (`.then(step)` chain) is synchronous-step-oriented; dynamic fan-out requiring parallel agent invocations must use Mastra's parallel step API, which the team has not yet used
- Debugging complex agent failures in Mastra requires Mastra Studio or structured log parsing — no fast iteration loop available without that infrastructure
- Tying walking skeleton learnings to Mastra's API surface makes it harder to reason about what is agent design vs. framework behaviour

**Rejection rationale:** Risk concentration. If the agent design is wrong AND the framework is unvalidated, debugging failures is ambiguous. Decouple those risks.

### Alternative 2: Pure LangChain.js / LangGraph

**Pros:**
- Mature, widely documented, large community
- LangGraph provides graph-based orchestration that maps well to the correlation/triage/hypothesis pipeline
- TypeScript SDK available

**Cons:**
- Third-party dependency outside the Claude Code ecosystem; requires LLM API calls via the Anthropic SDK rather than through Claude Code's built-in agent model
- Does not leverage the Claude Code Agent / Skill / Workflow primitives the user explicitly requested
- Introduces a new framework dependency when Claude Code provides equivalent primitives natively
- License: MIT (acceptable), but community support is Python-first; TypeScript SDK is secondary

**Rejection rationale:** Does not satisfy the user's explicit requirement to use Claude Code primitives. Also introduces framework coupling without the Mastra porting benefit.

### Alternative 3: Plain TypeScript orchestration with Anthropic SDK calls

**Pros:**
- Zero framework dependency
- Maximum control over LLM invocation, context assembly, and output parsing
- Easiest to audit and reason about

**Cons:**
- Requires reimplementing what Claude Code Agents and Workflows provide: agent invocation, structured output, parallel fan-out, error propagation
- No built-in skill injection mechanism — must build prompt assembly from scratch
- Walking skeleton becomes framework code, not domain code — misallocates the spike's learning value

**Rejection rationale:** Reimplements primitives that the Claude Code SDK already provides. Increases implementation cost for the walking skeleton without producing validated domain agent designs.

---

## Consequences

### Positive
- Walking skeleton validates agent decomposition and ODI coverage before any Mastra framework risk is incurred
- Claude Code CLI provides fast iteration: run a workflow, inspect structured output, revise agent instructions, re-run — all from the terminal
- Skills as `.md` files in `.claude/skills/` are framework-agnostic domain knowledge; they port to Mastra as agent instruction context verbatim
- Agent definitions in `.claude/agents/` declare explicit tool lists, making the port to Mastra's `tools:` config mechanical

### Negative
- Two-phase implementation: walking skeleton phase + Mastra port phase. Each phase requires effort; this decision accepts that cost as worthwhile for risk reduction
- Claude Code dynamic Workflows (JS orchestration scripts) are not TypeScript-typed; structured output validation must be done explicitly with JSON Schema
- The walking skeleton cannot serve real production OTel traffic without additional infrastructure wiring; it validates design logic, not operational readiness

### Porting Path (Claude Code → Mastra)

| Claude Code Primitive | Mastra Equivalent |
|-----------------------|-------------------|
| `.claude/agents/{name}.md` | `new Agent({ id, instructions, tools, model })` |
| `.claude/skills/{name}/SKILL.md` | Agent `instructions` + `context` field or memory injection |
| Workflow JS script (`.claude/workflows/`) | `createWorkflow().then(step)` with parallel step support |
| `Agent` tool invocation in workflow | `mastra.getAgent(id).generate(prompt)` |
| JSON Schema structured output | Zod schema on `outputSchema` per step |
| Bash tool in agent | Mastra `createTool({ execute: async () => { ... } })` |

---

## Enforcement

- Walking skeleton agents MUST declare `model: claude-sonnet-4-6` explicitly (no silent default inheritance) so model-level behaviour is identifiable during debugging
- Workflows MUST emit structured JSON matching the declared output schema; agents returning free-form text where JSON is expected are a red flag caught by the Workflow output validator
- Every agent definition MUST include a `probe()` section in its system prompt describing what it should do if a dependency (OTel endpoint, runbook filesystem path) is unavailable — this is the Earned Trust invariant for the walking skeleton stage
