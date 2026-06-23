# Options (Raw) — sre-agent-mttr-optimizer

Wave: DIVERGE
Phase: 3 — Brainstorming
Date: 2026-06-23
Skill: nw-brainstorming

---

## HMW Question

**How might we give on-call SREs a complete, evidence-backed diagnostic orientation the moment a production alert fires — before they have to ask for it?**

Validation against HMW rules:
- No embedded solution: PASS (no "dashboard," "agent," "Slack," or specific mechanism mentioned)
- Outcome-oriented: PASS (orientation + evidence, not a feature)
- Broad enough for genuinely different approaches: PASS (permits proactive vs. reactive, silent vs. conversational, read-only vs. agentic, terminal vs. Slack vs. embedded)
- Positive framing: PASS ("give SREs orientation," not "prevent SREs from being lost")

---

## SCAMPER Generation

*No evaluation language in this section. All ideas generated before any scoring.*

### S — Substitute: What if the core mechanism were replaced entirely?

**Current mechanism**: SRE manually queries 3–5 tools (APM, logs, metrics, runbook wiki) to build mental model.

**Substitution**: Replace the SRE-as-correlation-engine with an LLM agent that ingests raw OTel signals and generates the mental model automatically. The substitution is not "better tooling" — it is replacing the human cognitive act entirely with a machine cognitive act.

**Generated Option S1 — "The Incident Brief"**: An autonomous agent that activates on PagerDuty webhook, ingests the 60-second window of OTel traces, metrics, and logs around the alert, runs LLM reasoning over the correlated signals, and delivers a structured "incident brief" to the on-call SRE before they open their laptop. The brief format: 3-paragraph structure — (1) what is broken, (2) likely root cause with evidence citations, (3) immediate recommended action. No dashboard. No query required. One artifact, delivered.

**SCAMPER origin**: S (Substitute — replace human correlation with machine correlation as the primary mechanism)
**Closest competitor**: Datadog Bits AI SRE (autonomous investigation); Incident.io AI SRE

---

### C — Combine: What if this job were merged with an adjacent job?

**Adjacent job**: Post-mortem timeline reconstruction (currently 2–4 hour manual effort, per DISCOVER I-05 and I-10). Both jobs share the same inputs: OTel signals + alert events + runbook actions.

**Generated Option C1 — "Continuous Incident Scribe"**: A single agent that runs from alert-fires to post-mortem-closed. During the incident, it generates the diagnostic brief, logs all SRE actions taken, auto-narrates the "story so far" as a live timeline. After resolution, it auto-drafts the post-mortem timeline from the continuous event log — zero reconstruction effort. The live incident experience and the post-mortem artifact are produced by the same agent from the same evidence.

**SCAMPER origin**: C (Combine — merge live investigation and post-mortem reconstruction into a single continuous agent)
**Closest competitor**: Incident.io (closest to this model with AI Scribe on video calls + investigation); no product currently spans alert-to-post-mortem as a single continuous agent

---

### A — Adapt: What works well in a different domain? Could we borrow it?

**Domain to borrow from**: Medical diagnosis. A physician receiving a patient in acute distress follows a structured differential diagnosis protocol: (1) rule out the most dangerous explanation first, (2) test hypotheses against available diagnostic signals, (3) communicate certainty + uncertainty explicitly.

**Generated Option A1 — "Differential Diagnosis Protocol"**: A Socratic-style agent interaction model. Instead of delivering a briefing document, the agent asks the SRE one question at a time: "Is the error rate elevated on all services or just checkout?" Based on the answer, it narrows the hypothesis space using a decision tree derived from historical incident patterns. The SRE provides three data points via response; the agent delivers a ranked hypothesis at confidence ≥70% or escalates to "insufficient signal." The key adaptation: explicit uncertainty quantification, just as a physician communicates "90% confident this is X, 10% chance it's Y."

**SCAMPER origin**: A (Adapt — borrow differential diagnosis protocol from clinical medicine)
**Closest competitor**: None closely; Honeycomb BubbleUp is closest in concept (statistical differentiation) but requires human query construction

---

### M — Modify/Magnify: What if the most important dimension were amplified to the extreme?

**Most important dimension**: Speed. ODI-01 scores 16.0 — the highest under-served outcome. Every architecture decision should be evaluated against: does this add or subtract seconds from time-to-hypothesis?

**Generated Option M1 — "Zero-Latency Hypothesis Cache"**: An agent that does not wait for an alert — it pre-generates incident hypotheses continuously, 24/7, by running anomaly detection over OTel signals and pre-computing "if this anomaly pattern fires as an alert, the most likely causes are X, Y, Z with these supporting signals." When the alert actually fires, the hypothesis is already computed. The SRE receives a hypothesis that is not "generated in 60 seconds" but "delivered in <2 seconds" because it was pre-computed. The agent maintains a rolling hypothesis cache, updating continuously. Novel anomalies that don't match pre-computed patterns trigger an LLM reasoning path as fallback.

**SCAMPER origin**: M (Magnify — amplify speed to the extreme; pre-compute hypotheses before the alert fires)
**Closest competitor**: Dynatrace Davis AI (topology-based continuous monitoring with near-instant Problem generation); no LLM-native equivalent

---

### P — Put to other use: Who else has this job? Could the solution serve them too?

**Other stakeholders with the same job**:
- Incident Commanders (P-02 persona): need the same diagnostic orientation to coordinate responders, but consume it differently — they need a briefing they can narrate on a bridge call, not a technical trace analysis
- Platform engineering teams: running proactive chaos exercises need the same signal correlation to validate fault injection hypotheses
- SRE leadership reviewing post-mortems: need to understand incident patterns across dozens of incidents

**Generated Option P1 — "Multi-Persona Incident Hub"**: An agent that generates a primary technical brief (SRE-grade: trace IDs, span latencies, SQL queries) and simultaneously produces secondary artifacts: a "Commander Brief" (narrative prose, no technical jargon, suitable for bridge call narration) and a "Leadership Summary" (MTTR, blast radius, recurring pattern flag). The same OTel signals feed three differently formatted outputs for three different consumers in the same incident. Platform and runbook context are layered on top of the base OTel analysis.

**SCAMPER origin**: P (Put to other use — extend the diagnostic artifact to Incident Commander and SRE leadership personas)
**Closest competitor**: Incident.io's multi-role incident management; no product currently auto-segments the same investigation into persona-specific formats

---

### E — Eliminate: What if we removed the most complex part?

**Most complex part of the GIC job**: LLM-based reasoning over raw OTel signals at scale. Every vector of complexity in the current competitive landscape derives from the need to make AI reasoning over telemetry (a) fast enough, (b) accurate enough, and (c) explainable enough for production incident use.

**Radical question**: What if the agent never sees raw OTel signals at all?

**Generated Option E1 — "Runbook-First Hypothesis Matcher"**: Eliminate LLM-based signal reasoning entirely. Instead: an agent with exhaustive access to the org's runbooks and past incident post-mortems. When an alert fires, the agent performs semantic matching between the alert payload (service name, error code, affected endpoint) and the library of past incidents — returning "this matches Incident #342 (payment-gateway timeout, root cause: downstream rate limiting, resolution: verify rate limit config on redis-gateway). Confidence: 78%." No trace analysis. No metrics correlation. Pure pattern-matching against institutional knowledge. The hypothesis is "this looks like something that happened before" — which is exactly how senior SREs form first hypotheses mentally.

**SCAMPER origin**: E (Eliminate — remove LLM-based signal reasoning; rely on institutional memory matching)
**Closest competitor**: PagerDuty Outlier Incident (closest in concept — surfaces past similar incidents); BigPanda historical correlation; no product that specifically surfaces semantic matches to post-mortem knowledge bases

---

### R — Reverse: What if the workflow ran backwards? User and system switched roles?

**Current flow**: Alert fires → SRE wakes up → SRE investigates → SRE forms hypothesis.

**Reversed flow**: Agent continuously monitors → Agent forms hypothesis pre-alert → Agent wakes SRE only when hypothesis confidence is ≥X% → SRE receives hypothesis before they even read the alert title.

**Or reverse the information flow**: Instead of the SRE querying the observability stack, the observability stack pushes diagnostic context to wherever the SRE is already working — IDE, terminal, Slack — without any action required from the SRE.

**Generated Option R1 — "Push-to-Context Agent"**: Eliminate the concept of an "incident tool" entirely. The agent lives in the SRE's existing workflow context: a terminal plugin that intercepts kubectl, curl, and ssh commands during an incident and annotates them with real-time context ("the pod you're connecting to is in the blast radius — here is the hypothesis for why it's failing"). Or a VS Code extension that highlights the code last changed before the incident. The SRE never navigates to a tool — the diagnostic context follows them. The agent "pushes" investigation context to wherever the SRE's cursor is.

**SCAMPER origin**: R (Reverse — invert the flow from SRE-goes-to-tool to tool-follows-SRE)
**Closest competitor**: GitHub Copilot workspace integration (closest in mechanic — code context in IDE); no product that pushes OTel-based incident context into the SRE's terminal workflow

---

## Crazy 8s Supplements

*Additional options generated under 1-minute-per-idea constraint. Each must differ structurally from all SCAMPER options.*

### Supplement CS1 — "The Silent Partner" (Ambient Monitoring Agent)

A self-hosted, always-on agent deployed alongside the OTel Collector that never speaks unless spoken to — but maintains a continuously updated "incident state" in memory. When queried via CLI (`sre ask "why is checkout slow?"`), it responds in <5 seconds because the answer is pre-computed from the continuous monitoring context. The deployment model is: customer's own infrastructure, no SaaS dependency, no data leaves the perimeter. Runs as a sidecar to the OTel Collector. This is the on-prem/self-hosted antithesis of the SaaS-briefing model.

**Structural difference from SCAMPER options**: Deployment model (self-hosted sidecar to OTel Collector vs. SaaS agent). The OTel Collector as the deployment unit is distinct from all SCAMPER options, which assume either SaaS delivery or webhook delivery.

**Closest competitor**: No direct equivalent; OpenTelemetry Demo's built-in alertmanager is the closest reference point; conceptually similar to a local LLM deployment model

---

### Supplement CS2 — "The Chaos-First Agent" (Hypothesis Validation via Controlled Fault Injection)

Instead of passively analyzing signals from a live incident, this agent actively validates hypotheses by running controlled fault injection against non-production environments in parallel. When an alert fires on production suggesting "payment service timeout caused by downstream auth service degradation," the agent simultaneously re-creates the fault on staging, confirms whether the auth service degradation reproduces the payment timeout, and returns "confirmed — reproduced on staging within 90 seconds" as hypothesis validation evidence. The SRE receives not just a hypothesis but experimental confirmation.

**Structural difference from SCAMPER options**: Active hypothesis validation via parallel environment fault injection vs. passive signal analysis. Unique mechanism — no SCAMPER option does this.

**Closest competitor**: No direct equivalent in incident response; chaos engineering tools (LitmusChaos, Gremlin) perform fault injection but not as real-time hypothesis validation during live incidents

---

## Option Curation — Converging to 6

### All Generated Options

| ID | Name | Origin |
|----|------|--------|
| S1 | The Incident Brief | SCAMPER S |
| C1 | Continuous Incident Scribe | SCAMPER C |
| A1 | Differential Diagnosis Protocol | SCAMPER A |
| M1 | Zero-Latency Hypothesis Cache | SCAMPER M |
| P1 | Multi-Persona Incident Hub | SCAMPER P |
| E1 | Runbook-First Hypothesis Matcher | SCAMPER E |
| R1 | Push-to-Context Agent | SCAMPER R |
| CS1 | The Silent Partner (self-hosted) | Crazy 8s |
| CS2 | Chaos-First Hypothesis Validation | Crazy 8s |

Total: 9 options generated across 7 SCAMPER letters + 2 Crazy 8s.

### Structural Similarity Analysis

**P1 (Multi-Persona Hub) vs. S1 (Incident Brief)**: Both deliver a briefing artifact. P1's core mechanism is multi-audience output segmentation; S1's is single-audience high-fidelity brief. Structural difference: output segmentation vs. depth-over-breadth. These are not variations in degree — they represent different fundamental assumptions (does the brief serve one persona with high depth, or three personas with format differentiation?). Keep both.

**M1 (Hypothesis Cache) vs. S1 (Incident Brief)**: Same output (a diagnostic hypothesis) but structurally different mechanism (pre-computed vs. on-demand) and different cost profile (continuous compute vs. burst compute at alert time). Different mechanism, different assumption, different cost. Keep both — but M1 and CS1 share an architectural similarity (continuous monitoring, pre-computed state). See below.

**M1 (Hypothesis Cache) vs. CS1 (Silent Partner)**: Both involve continuous pre-computation of incident context. M1 assumes SaaS delivery; CS1 assumes self-hosted sidecar. The mechanism and cost profiles differ meaningfully (cloud-scale pre-computation vs. local-resource sidecar). However, both share the same core assumption (pre-computation beats on-demand reasoning for latency). This is a variation in deployment model, not mechanism. Merge: keep CS1 (the self-hosted model is more structurally distinct given the existing SaaS market saturation). M1's speed insight is captured within CS1's framing.

**CS2 (Chaos-First) vs. all others**: Unique mechanism (active fault injection for hypothesis validation vs. passive signal analysis in all other options). Keep.

**E1 (Runbook-First) vs. all others**: Unique mechanism (semantic matching against institutional knowledge with no signal reasoning). Entirely different assumption (knowledge-retrieval vs. signal-analysis). Keep.

**A1 (Differential Diagnosis) vs. S1 (Incident Brief)**: Both reason over signals but differ in interaction model — S1 is a push (no interaction), A1 is conversational/Socratic (pull). Different mechanism (asynchronous push vs. synchronous Q&A), different assumption (can the agent form a hypothesis from signals alone, or does it need SRE input to narrow the space?), different UX. Keep both.

**R1 (Push-to-Context) vs. all others**: Unique mechanism (ambient context injection into existing SRE workflow tools vs. a separate incident artifact). Unique assumption (SRE never needs to change their tool; context comes to them). Keep.

After merging M1 into CS1, the curated 6 are:

| Option | Name | Core Mechanism | 
|--------|------|----------------|
| OPT-1 | The Incident Brief | Proactive push — alert triggers agent, agent delivers single structured brief |
| OPT-2 | Continuous Incident Scribe | Alert-to-post-mortem continuous agent — combines live investigation and incident documentation |
| OPT-3 | Differential Diagnosis | Socratic Q&A — agent narrows hypothesis space via structured dialogue |
| OPT-4 | Multi-Persona Incident Hub | Multi-audience output — same signals generate SRE technical brief + Commander brief + Leadership summary |
| OPT-5 | Runbook-First Hypothesis Matcher | Institutional memory — semantic matching against past incidents and runbooks, no signal reasoning |
| OPT-6 | Self-Hosted Silent Partner | On-prem sidecar — self-hosted OTel Collector sidecar with pre-computed context; CLI query interface |
| (Bonus) OPT-7 | Push-to-Context Agent | Workflow-embedded — incident context pushed to SRE's terminal/IDE; no separate tool |
| (Bonus) OPT-8 | Chaos-First Hypothesis Validation | Active validation — parallel fault injection to experimentally confirm hypotheses |

*Note: 8 options generated that pass the diversity test. Curating to 6 by eliminating OPT-7 (Push-to-Context) and OPT-8 (Chaos-First) as out-of-scope for MVP given feasibility concerns:*

- **OPT-7 (Push-to-Context)**: The terminal/IDE embedding mechanism requires deep integration with the SRE's local development environment and kubectl workflows. The investment to achieve reliable, cross-tool context injection is disproportionate to the MVP scope. The ghost-in-the-chair job is about orientation before action, not workflow annotation during action. Eliminated: premature for MVP feasibility.
- **OPT-8 (Chaos-First)**: Active fault injection in parallel environments during a live P0 is a novel capability with significant blast-radius risk if the staging environment is used to validate database or network changes. The trust and safety requirements (C-01: autonomous execution must be opt-in, reversible, audit-logged) make this an advanced-autonomy feature, not a first-product design direction. Eliminated: trust prerequisites not met at MVP.

**Curated 6 for taste evaluation:**

| Option | Name | SCAMPER Origin |
|--------|------|----------------|
| OPT-1 | The Incident Brief | S (Substitute) |
| OPT-2 | Continuous Incident Scribe | C (Combine) |
| OPT-3 | Differential Diagnosis Protocol | A (Adapt) |
| OPT-4 | Multi-Persona Incident Hub | P (Put to other use) |
| OPT-5 | Runbook-First Hypothesis Matcher | E (Eliminate) |
| OPT-6 | Self-Hosted Silent Partner | Crazy 8s (incorporating M — Magnify/pre-compute) |

---

## The Curated 6 — Full Descriptions

### OPT-1: The Incident Brief

**Core idea**: When PagerDuty fires, the agent automatically ingests the 60-second OTel window around the anomaly (traces, metrics, logs), runs LLM-based causal chain reasoning, and delivers a structured 3-part brief to Slack: (1) what is broken, (2) ranked hypotheses with specific signal citations, (3) recommended immediate action. The on-call SRE receives it before opening their laptop. No query required, no tool to navigate to.

**Key mechanism**: Event-driven webhook (PagerDuty → agent → OTel query → LLM reasoning → Slack push). Agent is reactive to alerts but proactive to the SRE — the brief arrives before the SRE asks for it.

**Key assumption**: The most valuable moment is the first 60 seconds after an alert fires. A high-quality brief at that moment eliminates the 10–30 minutes of manual correlation. The agent can reason sufficiently from OTel signals alone, without requiring conversational input.

**SCAMPER origin**: S (Substitute — replaces the SRE-as-correlation-engine with a machine-as-correlation-engine)

**Closest competitor**: Datadog Bits AI SRE; Incident.io AI SRE

---

### OPT-2: Continuous Incident Scribe

**Core idea**: A single agent that runs continuously from alert-fires to post-mortem-closed. During the incident: generates diagnostic brief, logs all agent actions and SRE responses, narrates the "story so far" as a live timeline. After resolution: auto-drafts the post-mortem timeline from the continuous event log. The live investigation and the post-mortem artifact are produced by the same agent from the same evidence — zero separate reconstruction effort.

**Key mechanism**: Stateful agent with persistent incident memory. Not a webhook trigger that fires once — a long-running process that maintains incident state from start to finish and produces two distinct artifacts (live brief + post-mortem draft) from a single evidence stream.

**Key assumption**: The post-mortem reconstruction pain (2–4 hours, per DISCOVER I-05 and I-10) is valuable enough to users that combining it with the live investigation job creates a meaningfully stronger product. Both jobs share the same evidence — the cost of a combined solution is incremental over the live-investigation-only solution.

**SCAMPER origin**: C (Combine — merges live investigation and post-mortem reconstruction into one continuous agent)

**Closest competitor**: No direct equivalent; Incident.io's AI Scribe (video call transcription only) is the closest reference; no product currently spans alert-to-post-mortem as a single continuous agent

---

### OPT-3: Differential Diagnosis Protocol

**Core idea**: Rather than delivering a complete brief unilaterally, the agent engages the SRE in a structured diagnostic dialogue at incident start. "Alert fired on payment-gateway: (1) Is the error rate elevated on all upstream services or just checkout? (2) Did any deployment happen in the last 2 hours?" The SRE provides 3 data points via Slack reply; the agent combines their context with OTel signals and delivers a ranked hypothesis at ≥70% confidence. Uncertainty is explicit: "Confidence 73% — most likely auth service rate limit; 18% — downstream DB connection pool exhaustion."

**Key mechanism**: Structured elicitation + signal analysis synthesis. The agent is not passive (waiting for a query) nor fully unilateral (delivering a brief without input). It elicits 3 targeted pieces of context from the SRE, combines with OTel analysis, and returns a higher-confidence hypothesis than pure signal analysis alone could produce.

**Key assumption**: The SRE has local context that OTel signals do not capture (recent config change they made manually, a customer complaint they just saw, a deploy they know happened 5 minutes before the alert). 3 targeted questions extract this context efficiently (≤90 seconds) and materially improve hypothesis quality. The cost is 3 interactions vs. a unilateral brief.

**SCAMPER origin**: A (Adapt — differential diagnosis protocol from clinical medicine)

**Closest competitor**: No direct equivalent; Honeycomb's conversational query interface is the closest analog but is not incident-triggered or hypothesis-oriented

---

### OPT-4: Multi-Persona Incident Hub

**Core idea**: The same OTel signals feed three simultaneously generated artifacts for three different consumers in the same incident: (1) SRE Technical Brief — trace IDs, span latencies, SQL queries, confidence scores; (2) Commander Brief — 3-sentence prose narrative suitable for bridge call narration, jargon-free; (3) Leadership Summary — MTTR estimate, blast radius, recurring pattern flag, cost impact estimate. All three are delivered to the same Slack incident channel, formatted as separate collapsible sections. The Incident Commander reads one section; the on-call SRE reads another.

**Key mechanism**: Single OTel reasoning pass with multi-audience rendering. The LLM reasoning over signals is shared; the output layer produces three different register and format outputs from the same reasoning. An additional dimension: runbook step suggestions are layered into the SRE Technical Brief from the runbook corpus.

**Key assumption**: The incident management pain (Incident Commander's job, P-02 persona) and the diagnostic pain (on-call SRE's job, P-01) are best served by a single unified product rather than separate tools. The two-persona design creates enough differentiated value to justify the additional rendering complexity. SREs and Commanders co-exist in the same Slack incident channel, so multi-format output in one place serves both simultaneously.

**SCAMPER origin**: P (Put to other use — extends the investigation artifact to Incident Commander and SRE leadership)

**Closest competitor**: Incident.io (closest multi-role incident management); no product currently auto-segments the same investigation into persona-specific formats in a single artifact

---

### OPT-5: Runbook-First Hypothesis Matcher

**Core idea**: Eliminate LLM-based signal reasoning entirely at the first hypothesis step. When an alert fires, the agent performs semantic search over the org's runbook corpus and past incident post-mortems. Returns: "This alert pattern matches 3 historical incidents — closest match is Incident #342 (payment-gateway timeout, root cause: downstream Redis rate limiting, resolved in 18 min). Confidence: 78%. Next: verify rate limit config on redis-gateway." If no match above 60% confidence: "No pattern match. Running signal analysis..." and falls back to OTel-based reasoning. First response in <5 seconds for known patterns; 60-second fallback for novel ones.

**Key mechanism**: Hybrid retrieval-first, reasoning-second architecture. Pattern matching against institutional knowledge is fast, cheap, and highly relevant for recurring incident types. LLM signal reasoning is reserved for novel incidents that have no historical match. This inversion exploits the fact that most P1s are not novel — they are variants of known failure patterns.

**Key assumption**: ≥50% of P1 incidents are variants of previously seen failure patterns. Senior SREs confirm this intuitively ("a junior on-call engineer is going to spend three times as long on a problem a senior would recognize in five minutes" — DISCOVER I-05). The runbook corpus and post-mortem library contain enough institutional knowledge to serve as the primary hypothesis source for the majority of incidents. Runbook format: Markdown-in-Git (C-05 satisfied).

**SCAMPER origin**: E (Eliminate — remove LLM-based signal reasoning; rely on institutional memory matching)

**Closest competitor**: PagerDuty Outlier Incident (closest in concept — surfaces past similar incidents); no product that specifically performs semantic search of post-mortem knowledge bases as primary hypothesis engine

---

### OPT-6: Self-Hosted Silent Partner

**Core idea**: A self-hosted agent deployed as a sidecar to the customer's OTel Collector. Runs continuously, maintaining a rolling incident state model from the live OTel signal stream (not triggered by alerts). SREs query it via a CLI tool during incidents: `sre ask "why is checkout slow?"` — response in <5 seconds because context is pre-computed from continuous monitoring. Zero data leaves the customer's infrastructure perimeter. The deployment model is a single Docker container alongside the existing OTel Collector; configuration is a YAML file pointing to OTel backends (Prometheus, Loki, Tempo). The "silent" aspect: it does nothing visible until queried; it builds its understanding invisibly in the background.

**Key mechanism**: Continuous pre-computation of incident context, self-hosted deployment, CLI-native interface. The agent never "fires" proactively — it waits for the SRE's natural language question. But because context is pre-computed, the response is near-instant rather than requiring a 60-second LLM reasoning pass at query time. This model trades the proactiveness of OPT-1 for speed, on-prem data residency, and zero vendor lock-in.

**Key assumption**: For a significant segment of SRE teams (fintech, healthcare, enterprise with data residency requirements — confirmed in DISCOVER corpus: I-09 healthcare SaaS, I-01 payments platform), the ability to keep all telemetry data on-premises and behind the corporate firewall is a non-negotiable constraint. These teams would not adopt a SaaS briefing agent that reads their OTel traces, even with SOC2 certification. The self-hosted model is the only path to this customer segment.

**SCAMPER origin**: Crazy 8s (CS1) incorporating M (Magnify speed through pre-computation)

**Closest competitor**: No direct equivalent in the commercial market; conceptually analogous to a local LLM deployment model; technically similar to the OpenTelemetry Demo's built-in alertmanager patterns

---

## Diversity Test — All 6 Options

| Criterion | OPT-1 Brief | OPT-2 Scribe | OPT-3 Diagnosis | OPT-4 Hub | OPT-5 Runbook | OPT-6 Silent |
|-----------|-------------|--------------|-----------------|-----------|---------------|--------------|
| **Different mechanism?** | Alert-triggered LLM signal reasoning → push | Stateful long-running process → dual artifact | Structured elicitation + signal synthesis | Shared reasoning → multi-format render | Retrieval-first + reasoning fallback | Continuous pre-compute + CLI pull |
| **Different assumption about user behavior?** | SRE needs brief delivered before they ask | SRE values incident docs as much as live help | SRE context improves hypothesis quality | Two personas co-inhabit same tool | Most incidents repeat known patterns | SRE prefers to query tool on own terms |
| **Different cost/effort profile?** | Burst LLM cost at alert time; SaaS | Stateful process cost across incident lifetime | Lower LLM cost (targeted synthesis) | Higher rendering/LLM cost; multi-format | Cheapest LLM cost; retrieval-dominated | Infrastructure cost for continuous pre-compute; no SaaS fee |

All 6 options pass the 3-point diversity test. No two options share more than one of the three criteria.

---

## Eliminated Options

| Option | Reason Eliminated |
|--------|-------------------|
| OPT-7 (Push-to-Context / R1) | Terminal/IDE context injection requires deep local tool integration; out of scope for MVP feasibility. The ghost-in-the-chair job is about orientation before action, not during action. Revisit as v2 IDE integration. |
| OPT-8 (Chaos-First / CS2) | Active fault injection during live P0 violates C-01 (autonomous execution must be opt-in and reversible). Requires trust and safety infrastructure that is a prerequisite before this design direction is viable. Revisit as v3 autonomous validation feature. |
| M1 (Hypothesis Cache) | Merged into OPT-6 (Self-Hosted Silent Partner). Both share the continuous pre-computation mechanism; OPT-6's self-hosted deployment model is more structurally distinct and adds a new assumption vs. M1's SaaS variant. |

---

## Gate G3 Validation

| Criterion | Status | Evidence |
|-----------|--------|----------|
| 6 curated options | PASS | OPT-1 through OPT-6 |
| HMW question framed without embedded solution | PASS | No mechanism, tool, or interface specified in HMW |
| Each option passes diversity test (different mechanism, assumption, cost) | PASS | Diversity test table above — no two options share all three criteria |
| All 7 SCAMPER letters applied | PASS | S, C, A, M (merged into CS1), P, E, R (eliminated as OPT-7), plus 2 Crazy 8s |
| No evaluation language in options descriptions | PASS | No "better," "best," "innovative," "fails," or scoring language in options section |
| Eliminated options documented with rationale | PASS | Elimination table above |

**G3: PASS. Proceed to Phase 4 (Taste Evaluation).**
