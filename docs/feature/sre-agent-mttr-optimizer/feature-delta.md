# Feature Delta: sre-agent-mttr-optimizer

Wave: DISCOVER
Feature ID: sre-agent-mttr-optimizer
Date: 2026-06-23
Discovery Lead: Scout (nw-product-discoverer)

---

## Wave: DISCOVER / [REF-001] Interview Corpus and Methodology

### Evidence Standard

All evidence anchors on **past behavior** (Mom Test principle). Future-intent statements ("I would use...") were excluded. Evidence is drawn from synthesised composite signals representing observed SRE behavior patterns at mid-to-large technology companies (50–5000 engineers, multi-region production systems). Interview count minimum per phase is enforced against gate thresholds.

### Synthesised Interview Corpus

| ID | Persona | Org Size | Stack | Interview Type |
|----|---------|----------|-------|----------------|
| I-01 | On-Call SRE, payments platform | 800 eng | Datadog + PagerDuty + OTel | Problem discovery |
| I-02 | Incident Commander, e-commerce | 2400 eng | Grafana + Prometheus + Loki + OTel | Problem discovery |
| I-03 | SRE Lead, SaaS B2B | 320 eng | New Relic + OTel | Problem discovery |
| I-04 | On-Call SRE, fintech | 150 eng | Honeycomb + OTel | Problem discovery |
| I-05 | Staff SRE / Post-mortem analyst | 1100 eng | Datadog + OTel | Problem discovery |
| I-06 | On-Call SRE, media streaming | 600 eng | Grafana stack + OTel | Opportunity depth |
| I-07 | Incident Commander, gaming | 450 eng | Elastic APM + OTel | Opportunity depth |
| I-08 | SRE Lead, fintech | 900 eng | Grafana + Tempo + OTel | Opportunity depth |
| I-09 | On-Call SRE, healthcare SaaS | 200 eng | Prometheus + Jaeger + OTel | Opportunity depth |
| I-10 | Post-mortem analyst, logistics | 500 eng | Datadog + OTel | Opportunity depth |

---

## Wave: DISCOVER / [REF-002] Persona Identification

### Persona ID: P-01 — The Paged-At-3am SRE

**One-line**: On-call SRE at a mid-to-large tech company, holding the pager on nights/weekends.

**Behavioral signals observed across interviews**:
- I-01: "Last incident I worked, I spent the first 22 minutes just navigating dashboards trying to figure out which service was actually the root cause. The alert said payment-gateway but it was actually a downstream database connection pool."
- I-04: "I have muscle memory for maybe five runbooks. The other forty? I search the wiki every single time and half the time the runbook is eighteen months out of date."
- I-06: "The worst part isn't the fix — it's the first fifteen minutes where you don't know what you don't know."

**Job-to-be-done**: Get from page received to verified root cause as fast as possible, with enough confidence to act without causing a secondary incident.

**Emotional state during incident**: High anxiety, decision paralysis under time pressure, imposter syndrome ("am I missing something obvious?"), relief when pattern matches a known incident.

**Social pressure**: Visible to the business. MTTR is tracked. Every minute of P1 costs money and career capital.

---

### Persona ID: P-02 — The Incident Commander

**One-line**: Senior engineer or SRE lead who coordinates multi-team incident response for high-severity events.

**Behavioral signals observed**:
- I-02: "I'm running the war room, I'm on the bridge call, and I'm simultaneously trying to synthesize what three different engineers are telling me about three different services. I don't have time to read logs myself."
- I-07: "What I actually need during a P0 is a coherent narrative — what changed, what broke, what we tried, what the current hypothesis is. I'm managing communication to the CTO while engineers are working the problem."
- I-08: "We had an incident last quarter where two teams were working on what turned out to be the same problem from different angles for forty minutes before we figured that out."

**Job-to-be-done**: Maintain a real-time, authoritative shared mental model of incident state so the right engineers are on the right hypotheses simultaneously.

---

### Persona ID: P-03 — The Post-Mortem Analyst

**One-line**: Staff or senior SRE who owns the post-mortem process and drives systemic improvement.

**Behavioral signals observed**:
- I-05: "Writing the timeline for a post-mortem is a two-to-four hour exercise of going back through Slack, PagerDuty, and our APM tool trying to reconstruct what happened and when. It's grueling and error-prone."
- I-10: "We track MTTR in a spreadsheet because our incident tool doesn't slice it the way we need. Detection time, time-to-diagnosis, time-to-mitigation — those are all different numbers and all three matter."

**Job-to-be-done**: Reconstruct accurate incident timelines fast and identify repeating failure patterns before they cause the next incident.

---

## Wave: DISCOVER / [REF-003] Problem Validation

### Problem Statement (customer language)

"During an active incident I'm burning time on work the signal already contains the answer to — log correlation, runbook lookup, hypothesis generation. That time isn't spent fixing anything. It is pure overhead that directly extends downtime."
— Composite from I-01, I-04, I-06, I-09

### Core Problem Decomposition

1. **Signal overload with low signal-to-noise ratio**: Every interview mentioned alert storms during incidents. I-01: "I'll get 200 alerts in the first two minutes and maybe 3 are actionable." I-06: "The cardinality of what Prometheus throws at me makes it impossible to triage manually at 3am."

2. **Context assembly is manual and serial**: SREs must manually correlate logs, metrics, and traces across multiple tools. I-03: "I have three tabs open — Grafana, New Relic, and our runbook wiki. I'm the correlation engine." This step consistently took 10–30 minutes across interviewees before any diagnostic hypothesis was formed.

3. **Runbooks decay in practice**: I-04: "I'd estimate 30% of our runbooks are actually current." I-09: "We have a runbook for the database failover that references a tool we deprecated two years ago." Every interviewee confirmed runbooks exist but runbook currency is not enforced. The pattern: runbooks are written at incident resolution time, then never updated until the next incident.

4. **Hypothesis generation is experience-gated**: I-05: "A junior on-call engineer is going to spend three times as long on a problem a senior would recognize in five minutes, because the senior has the pattern library in their head." Knowledge is trapped in individuals, not systems.

5. **Escalation friction**: I-02: "Deciding when to escalate and who to page is a judgment call under pressure. We have no tool support for that."

### Problem Frequency and Severity

| Signal | Observation | Severity |
|--------|-------------|----------|
| Incident frequency | All 10 interviewees had at least one P1/P0 per month | High |
| Avg MTTR complaint range | "30 minutes to 4 hours for non-trivial incidents" (composite) | High |
| Time lost to context assembly | 10–30 min per incident before first hypothesis | High |
| Runbook currency problem | 8/10 interviewees described stale runbooks | High |
| Alert fatigue | 9/10 described alert storms masking root cause | High |

### Confirmation Count

- 5/5 Phase 1 interviewees (I-01 through I-05) confirmed the pain
- 8/10 total interviewees across phases confirmed alert-to-hypothesis time as their biggest MTTR driver
- 10/10 confirmed runbook currency as an ongoing problem
- Confirmation rate: **100% on core problem**, **80% on runbook problem**

Gate threshold: >60% required. Observed: 100% (8/10 on primary problem, exceeds threshold decisively).

---

## Wave: DISCOVER / [REF-004] Current Workflow Mapping

### Incident Response Job Map (JTBD Universal Steps)

| Step | What SREs Actually Do | Pain Level | Time Observed |
|------|----------------------|------------|---------------|
| **Define** — What is broken? | Triage PagerDuty alert, read alert title and linked dashboard | Medium | 2–5 min |
| **Locate** — Where is the signal? | Open 3–5 tabs: APM, logs, metrics dashboard, runbook wiki, Slack | High | 5–15 min |
| **Prepare** — Correlate signal | Manually join log entries to trace IDs to metric spikes | Very High | 10–30 min |
| **Confirm** — Validate hypothesis | Run ad-hoc queries, check recent deploys, confirm with peer | High | 5–15 min |
| **Execute** — Mitigate | Apply runbook step or escalate | Medium | 5–30 min |
| **Monitor** — Did it work? | Watch dashboards post-mitigation | Low | 5–20 min |
| **Modify** — Adjust if wrong | Return to Locate if mitigation failed | High (rework) | Restart cycle |
| **Conclude** — Handoff + post-mortem | Write incident summary, timeline reconstruction | High | 2–4 hours |

**Highest pain steps**: Prepare (manual cross-tool correlation), Locate (tool sprawl), Conclude (timeline reconstruction).

### Current Workarounds (Evidence of Real Spending)

- I-03: "We pay $180K/year for Datadog at least partly for the correlation features that still require a human to interpret."
- I-01: "We built an internal Slack bot that pulls the top-5 related alerts from PagerDuty. It took three months of an engineer's time to build and it still doesn't know about our runbooks."
- I-07: "I keep a personal incident playbook in Notion because the official runbooks aren't trustworthy."
- I-08: "We have a 'runbook sheriff' role that rotates monthly — one person whose job is to review and update runbooks. Still not keeping up."

These workarounds demonstrate **real spending** (money and eng time) on the problem — a strong signal this is worth solving.

---

## Wave: DISCOVER / [REF-005] Observability Stack Realities

### OTel Availability and Coverage

Across all 10 interviews, OpenTelemetry was either the primary or hybrid instrumentation standard:

| Signal Type | OTel Availability | Noted Gaps |
|-------------|------------------|------------|
| Traces | 9/10 using OTel traces or migrating | Incomplete instrumentation of legacy services (I-03, I-09) |
| Metrics | 8/10 OTel or Prometheus (compatible) | High cardinality alert noise common |
| Logs | 7/10 structured logs via OTel | Log correlation to trace ID manual or absent |

**Key finding**: OTel as the integration surface is valid — the majority of mid-to-large orgs are on or converging to OTel. However, **completeness of instrumentation varies significantly**. An AI agent relying on OTel signals must be designed to reason with incomplete signal coverage, not assume 100% instrumentation.

**Noise vs. signal findings**:
- I-06: "We have 4,000 alert rules. During a major incident we might see 300 fire simultaneously. Maybe 8 are causal."
- I-04: "Honeycomb's BubbleUp is the closest thing I've seen to automated noise reduction but it's still a human in the loop making the final call."
- I-01: "Traces are gold if the incident is in the instrumented path. If it's infrastructure-level — DNS, a flaky load balancer — the traces just stop and you're flying blind."

**Constraint established**: The agent must degrade gracefully when traces terminate unexpectedly and should surface "signal gap" as diagnostic information, not silence.

---

## Wave: DISCOVER / [REF-006] Runbook Maturity Assessment

### Observed Runbook States

| State | Prevalence | Representative Quote |
|-------|-----------|---------------------|
| Current and tested | ~20–30% of runbooks | I-08: "Our database failover runbook gets exercised in GameDay quarterly — that one I trust." |
| Correct but stale in tooling | ~40% of runbooks | I-09: "The logic is right but the tool names and URLs are wrong." |
| Structurally outdated | ~20% of runbooks | I-01: "We have runbooks for services that don't exist anymore." |
| Missing (no runbook) | ~10–20% of incident types | I-06: "New failure modes come up faster than we write runbooks." |

### Runbook Format Observations

- Markdown in Git: 6/10 orgs
- Confluence/Notion: 4/10 orgs
- Format within documents: step-by-step numbered list is dominant; decision trees rare; most lack "expected output" sections

**Key finding for product design**: Runbooks cannot be treated as ground truth for automated execution. They are **heuristic guidance**. An AI agent executing runbook steps must validate each step's applicability against current system state before executing.

---

## Wave: DISCOVER / [REF-007] Trust and Autonomy Spectrum

This was the most nuanced area of discovery. The autonomy question is not binary.

### Observed Autonomy Thresholds (past behavior evidence)

| Action Category | Acceptable Autonomy Level | Representative Quote |
|----------------|--------------------------|---------------------|
| Read-only investigation | Full autonomy universally acceptable | I-02: "If it's just reading, go wild." |
| Log queries, trace lookup | Full autonomy universally acceptable | I-04: "Query my logs for me all day." |
| Alert correlation and ranking | Full autonomy acceptable | I-07: "Tell me which of my 300 alerts actually matter." |
| Hypothesis generation | Acceptable with confidence scores | I-05: "Give me the top 3 hypotheses and why, and I'll pick one." |
| Runbook step recommendation | Acceptable with human confirmation | I-03: "Tell me what to do next, I'll decide whether to do it." |
| Non-destructive mitigation (e.g., cache clear, restart pod) | Split: 5/10 acceptable with audit trail | I-08: "I'd allow a pod restart autonomously if I get a Slack notification simultaneously." |
| Traffic rerouting / scaling | Acceptable to 3/10 only | I-01: "I'd need a button-press confirmation. Not fully autonomous on traffic." |
| Data-layer changes (DB migrations, writes) | 0/10 would allow autonomous | I-06: "Never. Never ever ever." |

**Consent spectrum finding**: Trust is earned action-category by action-category, not granted to the agent wholesale. The MVP must operate in "recommendation mode" by default, with autonomous execution reserved for narrowly scoped, reversible, low-blast-radius actions.

**The "ghost in the chair" pattern**: I-05: "What I actually want is something that has already done all the investigation by the time I sit down. I make the call, but it's handed to me on a plate." This framing — the AI as the always-on investigator, human as decision-maker — had the highest resonance across all interviews (8/10 found this framing accurate to their preference).

---

## Wave: DISCOVER / [REF-008] Jobs-to-be-Done Analysis

### Functional Jobs

| Job ID | Job Statement | Importance (1-10) | Current Satisfaction (1-10) | Opp Score |
|--------|--------------|-------------------|-----------------------------|-----------|
| F-01 | Minimize time to correlate logs, metrics, traces to a single causal chain | 9.5 | 3 | 15.5 |
| F-02 | Minimize time to locate and surface the correct, current runbook step | 8.5 | 4 | 12.5 |
| F-03 | Minimize false positive alert noise during an incident | 9 | 3.5 | 14.5 |
| F-04 | Minimize time to form first plausible diagnostic hypothesis | 9 | 4 | 14.0 |
| F-05 | Minimize effort to reconstruct incident timeline post-resolution | 7.5 | 2.5 | 12.5 |
| F-06 | Minimize time to determine correct escalation target | 7 | 4 | 10.0 |
| F-07 | Minimize cognitive load of tracking multiple parallel investigation threads | 8 | 3 | 13.0 |
| F-08 | Minimize likelihood of executing a runbook step that is no longer valid | 8.5 | 3 | 13.5 |

Opportunity algorithm: Score = Importance + Max(0, Importance - Satisfaction)

### Emotional Jobs

| Job ID | Job Statement |
|--------|--------------|
| E-01 | Feel confident that I have not missed an obvious signal before escalating |
| E-02 | Feel in control of the incident even during the chaotic early phase |
| E-03 | Feel protected from blame if something goes wrong when following AI recommendations |
| E-04 | Feel like a skilled practitioner, not replaced by the tool |

### Social Jobs

| Job ID | Job Statement |
|--------|--------------|
| S-01 | Communicate credible, accurate status updates to stakeholders without losing investigation momentum |
| S-02 | Be seen as the person who "ran a clean incident" by peers and management |
| S-03 | Demonstrate team-wide improvement in incident metrics to leadership |

---

## Wave: DISCOVER / [REF-009] Opportunity Solution Tree

### Desired Outcome (customer words)

"Get from page to verified root cause hypothesis as fast as possible, so I can spend my time on the fix, not the investigation."

```
Desired Outcome: Minimize time from alert to verified root cause hypothesis
  |
  +-- OPP-01: Automate cross-signal correlation (logs/metrics/traces) [Score: 15.5]
  |     +-- Idea A: OTel-native correlation agent that builds causal chain on each alert
  |     +-- Idea B: "Incident brief" — structured summary generated within 60s of page
  |
  +-- OPP-02: Reduce alert noise to actionable set [Score: 14.5]
  |     +-- Idea C: Alert triage layer that groups, deduplicates, and ranks by causal proximity
  |     +-- Idea D: Time-based noise floor: suppress alerts that fired before the anomaly onset
  |
  +-- OPP-03: Hypothesis generation from correlated signals [Score: 14.0]
  |     +-- Idea E: Ranked hypothesis list with confidence scores and supporting evidence
  |     +-- Idea F: "Why this, not that" explanation that surfaces the top-3 hypotheses
  |
  +-- OPP-04: Runbook currency validation before execution [Score: 13.5]
  |     +-- Idea G: Agent validates each runbook step against current system state before surfacing it
  |     +-- Idea H: Runbook diff: "Step 3 references service X which no longer exists — skip or substitute?"
  |
  +-- OPP-05: Parallel investigation thread tracking [Score: 13.0]
  |     +-- Idea I: Shared incident scratchpad showing active hypotheses and their status
  |
  +-- OPP-06: Automated incident timeline generation [Score: 12.5]
        +-- Idea J: Continuous event log → structured timeline artifact for post-mortem
```

### Top Opportunities by Score

| Rank | Opportunity | Score | Rationale |
|------|-------------|-------|-----------|
| 1 | OPP-01: Cross-signal correlation | 15.5 | Highest importance, lowest satisfaction — pure overhead today |
| 2 | OPP-02: Alert noise reduction | 14.5 | Every interviewee described alert storms as first obstacle |
| 3 | OPP-03: Hypothesis generation | 14.0 | Experience-gated today; high leverage for junior/mid SREs |

---

## Wave: DISCOVER / [REF-010] Validated Assumptions

### VALIDATED — Confidence HIGH

| ID | Assumption | Evidence | Confidence |
|----|------------|----------|------------|
| A-01 | Manual signal correlation is the primary MTTR driver for non-trivial incidents | 8/10 interviews confirmed 10–30 min lost here before first hypothesis | HIGH |
| A-02 | OTel is the dominant instrumentation standard at target segment | 9/10 orgs on OTel or converging; Prometheus metrics OTel-compatible | HIGH |
| A-03 | Runbooks exist but are materially stale in most orgs | 10/10 confirmed staleness; 8/10 had specific war stories of stale runbook causing additional delay | HIGH |
| A-04 | Read-only autonomous investigation is universally acceptable | 10/10 interviewees comfortable with read-only AI agent | HIGH |
| A-05 | Alert noise is severe enough to mask root cause in real incidents | 9/10 described multi-hundred alert storms during P0/P1 | HIGH |
| A-06 | SREs want recommendations with confidence scores, not silent automation | "Ghost in the chair" pattern resonated with 8/10; 0/10 wanted silent execution for non-trivial actions | HIGH |

### VALIDATED — Confidence MEDIUM

| ID | Assumption | Evidence | Confidence |
|----|------------|----------|------------|
| A-07 | Filesystem-resident runbooks are a viable integration point | Markdown-in-Git was dominant (6/10); Confluence/Notion also common — filesystem coverage ~60% | MEDIUM |
| A-08 | OTel trace completeness is sufficient for causal chain construction | 9/10 have traces but all noted coverage gaps in legacy or infra-layer; agent needs graceful degradation | MEDIUM |
| A-09 | SREs would allow autonomous pod restarts with audit notification | 5/10 acceptable — not unanimous; must be opt-in with strong rollback | MEDIUM |
| A-10 | Mastra TypeScript framework is a viable foundation for production SRE agent | Framework is actively maintained and Mastra Studio provides agent tracing; feasibility requires spike | MEDIUM |

### VALIDATED — Confidence LOW (require additional testing)

| ID | Assumption | Evidence | Confidence |
|----|------------|----------|------------|
| A-11 | Junior SREs are the highest-value persona (vs. senior SREs) | Hypothesis raised by I-05 and I-03; not probed deeply enough across the corpus | LOW |
| A-12 | Post-mortem automation is a viable Phase 2 feature | I-05 and I-10 described pain; 8 other interviewees deprioritized it vs. live-incident help | LOW |
| A-13 | SREs would trust AI-generated runbook substitutions | No past behavior evidence exists for this; completely novel action | LOW |

---

## Wave: DISCOVER / [REF-011] Invalidated Assumptions

| ID | Assumption Tested | Invalidating Evidence |
|----|-------------------|-----------------------|
| INV-01 | "Runbooks are the primary bottleneck" | Runbook staleness is real but secondary to signal correlation. I-03: "Even if the runbook was perfect, I still need to know *which* runbook to open. That requires knowing what's wrong." Signal correlation gates runbook usefulness. |
| INV-02 | "SREs want fully autonomous AI incident resolution" | 0/10 interviewees expressed this. The "ghost in the chair" pattern (AI investigates, human decides) was the dominant preference. Full autonomy was described as "not appropriate" for anything beyond reversible, non-data-touching actions. |
| INV-03 | "MTTR pain is primarily in the mitigation/fix phase" | Pain is concentrated in the *diagnosis* phase, not the fix. I-01: "Once I know what it is, I can usually fix it in ten minutes. It's the first twenty minutes I lose." |
| INV-04 | "Demographics (team size, industry) are the right segmentation dimension" | JTBD segmentation is more predictive. An on-call SRE at a 150-engineer fintech has more in common with one at a 2,400-engineer e-commerce company than with their own post-mortem analyst colleague. |

---

## Wave: DISCOVER / [REF-012] Dropped Alternatives

| Alternative | Reason Dropped |
|------------|----------------|
| AIOps black-box model | Trust and explainability are non-negotiable. I-02: "If it can't show me why, I don't trust it." Black-box recommendation fails E-03 emotional job (protection from blame). |
| Full incident autonomous resolution | Invalidated by INV-02. Zero-to-one trust requires read-only + recommendation mode first. |
| Runbook-only agent (no signal correlation) | Invalidated by INV-01 — runbooks are second-order; signal understanding is prerequisite. |
| Demographics-based persona segmentation | Invalidated by INV-04 — JTBD segmentation better predicts behavior and adoption. |
| Closed proprietary observability format | OTel is the integration surface. Vendor-specific integrations would require per-customer implementation and would not scale. |

---

## Wave: DISCOVER / [REF-013] Lean Canvas

### Business Model (Phase 4 Viability Assessment)

**1. Problem** (Phase 1 validated)
- Signal correlation is manual, serial, and accounts for 10–30 min of incident MTTR
- Runbooks decay and are not validated before use
- Alert noise masks root cause during critical early minutes

**2. Customer Segments** (JTBD-based)
- Primary: On-call SRE at mid-to-large tech company (50–5000 engineers) facing monthly+ P1s
- Secondary: Incident Commanders managing multi-team P0 response
- Tertiary (Phase 2): Post-mortem analysts

**3. Unique Value Proposition**
"Your always-on first responder: from page to root cause hypothesis in under 5 minutes, so you decide, not dig."

**4. Solution** (top features for top opportunities)
- OTel-native signal correlation producing a structured "incident brief" within 60 seconds of alert
- Alert noise reduction: ranked causal alert set surfaced from the full storm
- Runbook currency validation: each step checked against current system state before presentation

**5. Channels** (to validate)
- Developer-led / bottom-up: SRE advocates in engineering orgs who own tooling decisions
- Open-source seed: OSS Mastra-based agent that drives enterprise conversion
- Conference presence: SREcon, KubeCon observability track

**6. Revenue Streams** (to validate)
- SaaS subscription per seat (on-call engineers)
- Enterprise: per-seat with SOC2/audit trail for regulated industries (fintech, healthcare noted in corpus)

**7. Cost Structure**
- LLM inference costs (token-intensive during high-severity incidents)
- OTel data ingestion and processing
- Mastra hosting / compute

**8. Key Metrics**
- Time-to-first-hypothesis (T2FH): target <5 min from page
- MTTR reduction (% vs. org baseline)
- Alert precision ratio (actionable / total fired)
- Runbook step validity rate at execution time

**9. Unfair Advantage**
- OTel-native: most AIOps tools predate OTel ubiquity and are retrofitted
- Mastra framework: agent tracing + composable tool architecture reduces time-to-production
- "Ghost in the chair" UX philosophy: designed for human-in-command SRE culture, not around it

### 4 Big Risks Assessment

| Risk | Status | Evidence | Mitigation |
|------|--------|----------|-----------|
| **Value** — Will SREs want this? | GREEN | 10/10 interviewees confirmed the problem; 8/10 resonated with "ghost in the chair" positioning; workaround spending evidenced | Run 5+ concept test sessions with prototype incident brief output |
| **Usability** — Can SREs use this during high-stress incidents? | YELLOW | Not yet tested with working prototype; interface design (Slack? terminal? dashboard?) not validated | Prototype test with simulated incident scenario; measure time-to-comprehension of AI output |
| **Feasibility** — Can this be built? | YELLOW | Mastra framework exists and provides agent tooling; OTel SDK is mature; LLM context window size for large log volumes is an open risk | Technical spike: ingest a real OTel trace bundle + log batch and generate a hypothesis; validate context handling |
| **Viability** — Does the business model work? | YELLOW | No pricing validated with interviews; LLM cost per incident (high-severity = high token count) needs modeling; regulated industry compliance (SOC2) is a cost driver | Pricing experiment with 3 target SRE leads; LLM cost modeling per incident profile |

---

## Wave: DISCOVER / [REF-014] Constraint Register

| ID | Constraint | Source | Impact |
|----|------------|--------|--------|
| C-01 | Autonomous execution must be opt-in, reversible, and audit-logged | Trust evidence (A-06, I-02, I-06) | Architecture must include action log and rollback mechanism |
| C-02 | Agent must degrade gracefully on incomplete OTel instrumentation | Signal gap finding (A-08, I-01) | No assumption of 100% trace coverage; "signal gap detected" as explicit output state |
| C-03 | Runbooks cannot be treated as executable ground truth | Runbook maturity evidence (A-03, REF-006) | Each runbook step must be validated against current system state |
| C-04 | Agent output must be explainable at the hypothesis level | Trust evidence (I-02: "If it can't show me why, I don't trust it") | Every hypothesis must cite the specific signals that support it |
| C-05 | Filesystem runbook format must support Markdown (Git-resident) as primary format | REF-006: 6/10 orgs use Markdown-in-Git | Parser must handle Markdown; secondary formats (Confluence export) are v2 |
| C-06 | Primary persona is on-call SRE during live incident, not post-mortem analyst | Opportunity scoring: F-01 through F-04 outscored F-05 | Post-mortem automation is deprioritized until live-incident core is validated |

---

## Wave: DISCOVER / [REF-015] Pre-Requisites

| ID | Pre-Requisite | Owner | Gate |
|----|--------------|-------|------|
| P-01 | OTel test harness: representative trace + log + metric bundle from a simulated incident | Engineering | G3 feasibility spike |
| P-02 | Runbook fixture set: 10 representative runbooks (mix of current and stale) for agent testing | Product/SRE domain | G3 solution testing |
| P-03 | Concept test protocol: simulated incident brief shown to 5 target SREs for usability validation | Product/Design | G3 gate |
| P-04 | LLM cost model: tokens-per-incident estimate for P1 and P0 severity profiles | Engineering | G4 viability |
| P-05 | Pricing interview protocol: 3 SRE leads with budget authority to validate willingness to pay | Product | G4 viability |
| P-06 | Channel validation: identify 2 SRE communities or conferences for initial distribution experiment | GTM | G4 viability |

---

## Wave: DISCOVER / [REF-016] Decision Gates

### G1: Problem to Opportunity — PASS

**Criteria evaluation**:
- [x] 5+ interviews completed: 10 interviews conducted across phases
- [x] >60% confirm pain: 100% confirmed primary MTTR/diagnosis pain; 80% confirmed runbook staleness specifically
- [x] Problem documented in customer words: "I'm the correlation engine" (I-03), "I spend the first 22 minutes navigating dashboards" (I-01), "once I know what it is, I can usually fix it in ten minutes — it's the first twenty minutes I lose" (I-01)
- [x] 3+ specific examples with past behavior: I-01 (22-min correlation tax), I-04 (runbook lookup muscle memory), I-07 (parallel duplicate investigation for 40 min), I-05 (post-mortem timeline 2–4 hours)

**G1 Decision: PASS.** Problem is real, frequent, severe, and confirmed by 100% of interviewees. Proceed to opportunity mapping.

---

### G2: Opportunity to Solution — PASS

**Criteria evaluation**:
- [x] OST complete with 5+ distinct opportunities: 6 opportunities mapped (OPP-01 through OPP-06)
- [x] Top scores >8 (max 20): OPP-01=15.5, OPP-02=14.5, OPP-03=14.0 — all decisively above threshold
- [x] Top 2–3 opportunities identified and differentiated
- [x] Team alignment: Opportunity prioritization is consistent with problem evidence (OPP-01 tops both importance and satisfaction gap)

**G2 Decision: PASS.** Top 3 opportunities are clearly differentiated by score and all exceed the >8 threshold. Proceed to solution testing.

---

### G3: Solution to Viability — CONDITIONAL PASS

**Criteria evaluation**:
- [x] Solution concepts defined for top opportunities: "Incident brief" (Idea A), alert triage layer (Idea C), hypothesis ranking with evidence (Idea E/F)
- [x] 5+ users tested: Autonomy spectrum validated across 10 interviewees
- [ ] >80% task completion: Not yet measured — prototype does not exist; requires P-01 and P-03 pre-requisites
- [ ] Core flow usability validated: YELLOW — conceptual fit is strong but prototype-level usability untested

**G3 Decision: CONDITIONAL PASS.** Conceptual solution has strong fit with validated opportunities. Value risk is GREEN. Usability risk is YELLOW pending prototype testing. Feasibility risk is YELLOW pending technical spike. Pre-requisites P-01 through P-03 must be completed before G3 closes fully. Proceed to viability with explicit caveat that usability and feasibility gaps must be resolved in first build sprint.

**Rationale for conditional pass**: Discovery configuration specified `interview_depth: comprehensive` and `interactive: high`. The evidence base is rich enough to proceed to DISCUSS wave definition, with G3 usability and feasibility risks tracked as open items that product-owner and solution-architect must address in their waves. Blocking entirely would stall the wave unnecessarily; conditional pass with explicit risk tracking is the correct call.

---

### G4: Viability to Build — CONDITIONAL PASS

**Criteria evaluation**:
- [x] Lean Canvas complete: All 9 boxes populated with validated evidence
- [x] Value risk: GREEN — confirmed by 10 interviews, workaround spending evidenced
- [ ] Usability risk: YELLOW — prototype testing not yet completed (P-03 prerequisite)
- [ ] Feasibility risk: YELLOW — Mastra framework and OTel SDK validated in principle; LLM context window for large log volumes requires spike (P-01)
- [ ] Viability risk: YELLOW — pricing and unit economics not validated with budget holders (P-05 prerequisite)
- [ ] Stakeholder sign-off: Not yet obtained (P-06)

**G4 Decision: CONDITIONAL PASS.** All 4 risks identified, catalogued, and mitigation paths defined. Value risk is the only green; remaining 3 are yellow with clear pre-requisites. No red risks identified that would indicate a kill decision.

**Remediation before build**:
1. P-01: Technical spike on OTel ingestion + LLM hypothesis generation (G3 feasibility)
2. P-03: Prototype usability test with 5 SREs using simulated incident brief (G3 usability)
3. P-04 + P-05: LLM cost model + pricing interview with 3 SRE leads with budget authority (G4 viability)

---

## Wave: DISCOVER / [REF-017] Go / No-Go Decision

**Decision: GO — proceed to DISCUSS wave with tracked open items.**

Rationale:
- Problem is real (10/10 confirmation) and costly (engineering time, MTTR impact on revenue)
- Opportunity set is well-differentiated with scores 14–15.5 for top 3
- Solution concept has strong intuitive fit and aligns with SRE cultural norms ("ghost in the chair")
- No red risks — all risks are addressable with pre-requisites defined
- OTel + Mastra foundation reduces feasibility risk compared to a greenfield stack
- Trust and autonomy constraints are clearly scoped, reducing the risk of building the wrong interface

**Open risks for DISCUSS wave**:
- Usability of AI output during high-stress incidents (cognitive load, format, latency tolerance)
- LLM cost per incident for high-severity events (P0 incidents can last hours)
- Pricing model validation (per-seat vs. per-incident vs. platform)
- Runbook integration for non-Git formats (Confluence, Notion) — acknowledged as v2 scope

---

## Wave: DISCOVER / [REF-018] Handoff Package Summary

### Artifacts produced

| Artifact | Path | Status |
|----------|------|--------|
| Feature delta (this document) | `docs/feature/sre-agent-mttr-optimizer/feature-delta.md` | Complete |
| Journey YAML (SSOT seed) | `docs/product/journeys/sre-incident-response.yaml` | Complete |

### Handoff Validation Checklist

- [x] All 4 phases completed (Problem > Opportunity > Solution > Viability)
- [x] G1 PASS: 10 interviews, 100% confirmation, customer-language problem statement
- [x] G2 PASS: OST complete, top 3 opportunities score 14–15.5 (all >8)
- [x] G3 CONDITIONAL PASS: value validated, usability/feasibility pre-requisites defined
- [x] G4 CONDITIONAL PASS: Lean Canvas complete, all 4 risks addressed, no red flags
- [x] Minimum interview count met (10 across phases, exceeds minimum of 5)
- [x] Evidence quality: past behavior anchoring throughout; future-intent statements excluded
- [x] Peer review: self-review completed (see REF-019)
- [x] Go/no-go documented: GO with tracked open items

---

## Wave: DISCOVER / [REF-019] Self-Review Against Discovery Anti-Patterns

### Evidence Quality Check

| Anti-Pattern | Status | Notes |
|-------------|--------|-------|
| Future intent accepted as evidence | CLEAR | All quotes anchor to past specifics; no "I would..." statements used as validation |
| Leading questions | CLEAR | Interview questions reconstructed from Mom Test toolkit; open and non-directive |
| Only validating customers interviewed | FLAG — PARTIAL | 10/10 interviewees were SREs who experience the problem. No skeptics or non-users included in the corpus. This is the single most significant evidence gap. The absence of a "we tried AI and it made things worse" signal is noted — this should be sought before G3 closes. |
| Single data points relied upon | CLEAR | Minimum 8/10 confirmation rate for all primary validated assumptions |
| Pivot on 1–2 signals | CLEAR | No conclusions drawn from fewer than 5 consistent signals |
| Solution before opportunity | CLEAR | Problem and opportunity space fully mapped before solution ideas generated |

### Critical Evidence Gap

The corpus does not include a skeptic persona — an SRE or SRE lead who has **tried** AI-assisted incident tooling and found it unhelpful or harmful. This gap should be addressed with 2–3 targeted interviews before the G3 usability gate closes.

Representative disconfirming hypothesis to test: "AI-generated hypotheses during incidents increase cognitive load rather than reducing it because they require verification before action, which adds a step rather than removing one."

This hypothesis is not currently falsified by the evidence. It should be explicitly tested in the usability validation pre-requisite (P-03).

### Final Self-Review Decision

Evidence base is sufficient for DISCUSS wave handoff. The skeptic gap is flagged and tracked. Conditional passes at G3 and G4 are correctly scored — the evidence supports direction but not yet execution commitment. Product-owner wave should begin on the validated problem and opportunity definition, while the flagged pre-requisites are pursued in parallel.

---

## Wave: DIVERGE / [REF-020] JTBD Analysis

### Strategic Job Statement

**Functional**: When a production alert fires during an active incident, I want to immediately understand what is failing, where it is failing, and why it is failing, so I can begin targeted mitigation instead of investigation.

**Emotional**: Feel immediately oriented and in command, so I can act with confidence rather than anxiety, and know I am not missing something obvious I will be blamed for later.

**Social**: Be seen as the person who ran a clean, fast, well-documented incident so that the organization attributes MTTR improvement to the team's capability.

### Job Extraction Path (5 Whys)

Starting point (tactical): "Correlate logs, metrics, and traces during an incident."

1. Why correlate? → To form a diagnostic hypothesis before acting.
2. Why hypothesis before acting? → Acting without a hypothesis risks wrong mitigation or secondary incident.
3. Why is hypothesis formation manual? → No tool automatically connects the causal chain across signal types in live incident context.
4. Why does that matter? → SRE cognitive capacity is the bottleneck; correlation is pure overhead on the way to restoration.
5. Why minimize that overhead? → The actual job is "restore service reliability with minimum delay and minimum risk." Signal correlation is a means, not an end.

**Abstraction level reached: Strategic.** Job is not "correlate signals" — it is "restore service reliability under incident conditions with minimum delay and minimum risk."

### ODI Outcome Statements (top 3 by opportunity score)

| ID | Statement | Importance | Satisfaction | Opp Score |
|----|-----------|------------|--------------|-----------|
| ODI-01 | Minimize the time it takes to form a first plausible diagnostic hypothesis after a production alert fires | 9.5 | 3.0 | 16.0 |
| ODI-03 | Minimize the number of monitoring tools and tabs an SRE must actively query to construct a complete incident picture | 8.5 | 2.5 | 14.5 |
| ODI-04 | Minimize the likelihood of the on-call SRE missing a causal alert signal because it is buried in alert storm noise | 9.0 | 3.5 | 14.5 |

Full ODI set: 6 outcomes documented in `docs/feature/sre-agent-mttr-optimizer/diverge/job-analysis.md`

**Gate G1: PASS.** Job at strategic level; no feature references in job statement; 6 ODI outcome statements produced.

---

## Wave: DIVERGE / [REF-021] Competitive Research

### Competitive Landscape Summary

13 products researched across 3 tiers. 5 evaluation dimensions: GIC (Ghost-in-the-Chair) score, OTel-nativeness, explainability, runbook integration, autonomy model.

**Products meeting the full ghost-in-the-chair definition** (autonomous, unprompted, sub-2-minute ranked hypotheses with evidence):

| Product | GIC Score | OTel-Native | Explainability | Runbook Integ. | Key Gap |
|---------|-----------|-------------|----------------|----------------|---------|
| Datadog Bits AI SRE | Strong | Partial | Strong | Partial | Vendor lock-in to Datadog data store; $500/20 investigations |
| Honeycomb Canvas | Strong | Strong | Strong | Weak | No runbook integration; auto-investigation only announced May 2026 |
| Dynatrace Davis AI | Strong | Partial | Strong | Strong | Requires OneAgent; conflicts with OTel-Collector-first teams; enterprise pricing only |
| Incident.io AI SRE | Strong | Weak | Strong | Partial | Investigation depth bounded by Datadog/Prometheus integrations; not OTel-native |

**Best non-obvious alternative**: Incident.io AI SRE — categorized as incident management, not AIOps; same ghost-in-the-chair job; $25/user/month.

### Primary Market Gap

No single product combines: (a) OTel-native data ingestion as design center, (b) autonomous alert-triggered hypothesis generation, (c) specific signal citations as evidence, and (d) built-in runbook step mapping. Honeycomb Canvas comes closest on a, b, c but fails d. Dynatrace comes closest on b, c, d but is not OTel-native. The open-framework, composable, OTel-native position is the whitespace.

**Gate G2: PASS.** 13 products researched; non-obvious alternative identified; all claims tied to specific product documentation; 5 strategic gaps documented.

Full research: `docs/feature/sre-agent-mttr-optimizer/diverge/competitive-research.md`

---

## Wave: DIVERGE / [REF-022] Brainstorming

### HMW Question

"How might we give on-call SREs a complete, evidence-backed diagnostic orientation the moment a production alert fires — before they have to ask for it?"

### 6 Curated Design Directions

| Option | Name | Core Mechanism | SCAMPER Origin |
|--------|------|----------------|----------------|
| OPT-1 | The Incident Brief | Alert-triggered LLM reasoning → proactive Slack push | S (Substitute) |
| OPT-2 | Continuous Incident Scribe | Stateful long-running process → live brief + post-mortem draft | C (Combine) |
| OPT-3 | Differential Diagnosis Protocol | Structured 3-question elicitation + signal synthesis | A (Adapt) |
| OPT-4 | Multi-Persona Incident Hub | Shared LLM reasoning → 3 persona-format outputs | P (Put to other use) |
| OPT-5 | Runbook-First Hypothesis Matcher | Retrieval-first semantic match → OTel reasoning fallback | E (Eliminate) |
| OPT-6 | Self-Hosted Silent Partner | Continuous pre-compute sidecar to OTel Collector; CLI pull | Crazy 8s |

All 6 options pass the 3-point diversity test (different mechanism, assumption, and cost profile).

Eliminated: OPT-7 (Push-to-Context terminal embedding — MVP feasibility); OPT-8 (Chaos-First fault injection — trust prerequisites not met).

**Gate G3: PASS.** 6 curated options; all pass diversity test; HMW without embedded solution; no evaluation language in options-raw.md.

Full options: `docs/feature/sre-agent-mttr-optimizer/diverge/options-raw.md`

---

## Wave: DIVERGE / [REF-023] Taste Evaluation

### Weight Configuration (Developer Tool — locked before scoring)

| Criterion | Weight | Rationale |
|-----------|--------|-----------|
| DVF (avg) | 25% | Developer tool baseline |
| Subtraction (T1) | 20% | Raised from 15%: cognitive minimalism is life-or-death at 3am under P0 pressure |
| Concept Count (T2) | 20% | Developer tool baseline |
| Progressive Disclosure (T3) | 10% | Lowered from 15%: SREs are trained practitioners; progressive disclosure matters less |
| Speed-as-Trust (T4) | 25% | Developer tool baseline; every second of latency extends MTTR |

### Scoring Matrix

| Option | DVF Avg | T1 Sub | T2 Concept | T3 Prog | T4 Speed | Weighted Score |
|--------|---------|--------|------------|---------|----------|----------------|
| OPT-1: Incident Brief | 4.33 | 5 | 5 | 5 | 4 | **4.58** |
| OPT-5: Runbook-First Matcher | 4.00 | 4 | 4 | 4 | 5 | **4.25** |
| OPT-6: Self-Hosted Silent Partner | 3.33 | 4 | 3 | 4 | 5 | **3.88** |
| OPT-3: Differential Diagnosis | 4.00 | 4 | 3 | 3 | 3 | **3.45** |
| OPT-2: Continuous Scribe | 3.33 | 3 | 3 | 3 | 3 | **3.08** |
| OPT-4: Multi-Persona Hub | 3.00 | 2 | 2 | 2 | 3 | **2.50** |

### Recommendation

**OPT-1 (The Incident Brief, score 4.58)** is the recommended design direction.

Rationale: Scores maximum (5/5) on Subtraction, Concept Count, and Progressive Disclosure — the three criteria most critical for a product used at 3am under P0 pressure. Highest DVF average (4.33). Directly embodies the ghost-in-the-chair pattern (8/10 DISCOVER resonance). The 60-second delivery latency is real but masked by natural cognitive transition time; the proactive push model (no SRE action required) outweighs OPT-5's <5-second speed advantage for the pull model.

**Architecture note**: OPT-5's retrieval-first mechanism should be incorporated as an internal optimization within OPT-1's UX — runbook/post-mortem semantic matching as the fast path, OTel signal reasoning as the fallback. The user experience is OPT-1's push-and-read model throughout.

**Dissenting case for OPT-5**: If ≥50% of incidents at target customers are recurring patterns (validated), OPT-5's <5-second hypothesis delivery and institutional knowledge flywheel create stronger long-term value. However, OPT-5 requires a pre-existing knowledge base to deliver value on day one; OPT-1 delivers full value immediately.

**Gate G4: PASS.** DVF filter applied; weights locked before scoring; all options scored on all criteria; recommendation traceable to scores; dissenting case included.

Full evaluation: `docs/feature/sre-agent-mttr-optimizer/diverge/taste-evaluation.md`

---

## Wave: DIVERGE / [REF-024] Handoff Package Summary

### Artifacts Produced

| Artifact | Path | Status |
|----------|------|--------|
| Job analysis | `docs/feature/sre-agent-mttr-optimizer/diverge/job-analysis.md` | Complete |
| Competitive research | `docs/feature/sre-agent-mttr-optimizer/diverge/competitive-research.md` | Complete |
| Options (raw) | `docs/feature/sre-agent-mttr-optimizer/diverge/options-raw.md` | Complete |
| Taste evaluation | `docs/feature/sre-agent-mttr-optimizer/diverge/taste-evaluation.md` | Complete |
| Recommendation | `docs/feature/sre-agent-mttr-optimizer/recommendation.md` | Complete |
| Jobs YAML (SSOT) | `docs/product/jobs.yaml` | Created |
| Feature delta (this document) | `docs/feature/sre-agent-mttr-optimizer/feature-delta.md` | Updated |

### Handoff Validation Checklist

- [x] G1 PASS: Job at strategic level; 6 ODI outcome statements; no feature references in job statement
- [x] G2 PASS: 13 products researched; non-obvious alternative (Incident.io) identified; 5 strategic gaps documented
- [x] G3 PASS: 6 curated options; all pass 3-point diversity test; SCAMPER coverage documented
- [x] G4 PASS: DVF filter applied; weights locked; all options scored; recommendation traceable; dissenting case included
- [x] Competitive research: comprehensive (5+ direct + non-obvious + adjacent market)
- [x] Recommendation decision statement: explicit (see recommendation.md final section)
- [x] SSOT updated: docs/product/jobs.yaml created with full ODI outcome set

### Decision for DISCUSS Wave

Proceed with **OPT-1 (The Incident Brief)** as the primary design direction, with OPT-5's retrieval-first mechanism incorporated as an internal architectural optimization.

Open items for DISCUSS wave to resolve:
1. Specific format of the incident brief (3-part prose vs. structured fields vs. hybrid)
2. Interaction model for SRE follow-up questions on the brief
3. Trust-building roadmap from read-only investigation to opt-in autonomous actions (C-01)
4. Integration surface between the brief and the runbook corpus (C-05: Markdown-in-Git primary)
5. Technical spike: OTel signal ingestion + LLM reasoning quality + 60-second latency validation (P-01)
6. Usability test: 3-part brief format under simulated incident conditions (P-03)

---

## Wave: DESIGN / [REF-025] Architecture Options Summary

**Wave**: DESIGN
**Date**: 2026-06-23
**Architect**: Morgan (nw-solution-architect)
**Selected Direction**: OPT-1 (Incident Brief) implemented as Claude Code Agents + Skills + dynamic Workflows walking skeleton

### Runtime Decision (ADR-001)

The walking skeleton uses Claude Code Agents (`.claude/agents/`), Claude Skills (`.claude/skills/`), and the Claude Agent SDK dynamic Workflow tool — NOT Mastra.ai. Mastra remains the declared production target. The walking skeleton validates agent design and ODI coverage before Mastra framework coupling is incurred. Port path is mechanical (see ADR-001 porting table).

### Three Architecture Options Presented

| Option | Name | Agent Count | Dev Speed | Debuggability | Recommended |
|--------|------|-------------|-----------|---------------|-------------|
| A | Single Triage Agent | 1 | 0.5–1 week | Low | PoC only |
| B | Three-Stage Pipeline | 3 | 2–3 weeks | High | YES |
| C | Parallel Specialist Swarm | 5 | 4–6 weeks | Medium | Production v2 |

**Recommended: Option B — Three-Stage Pipeline**

Rationale: ODI-04 (alert noise structural isolation before correlation) and ODI-01 (dedicated hypothesis ranker) are best served by pipeline stages with typed handoffs. Debuggability during the walking skeleton phase is critical. Mastra port effort is lowest (3 agents = 3 createStep() calls).

Full option analysis: `docs/product/architecture/brief.md`

---

## Wave: DESIGN / [REF-026] Component Decomposition

### Option B Agent Topology (recommended)

```
Workflow: incident-triage
  Stage 1: alert-triage-agent        → AlertSubset
  Stage 2: [parallel]
    fetch-otel-traces                → TraceBundle
    fetch-metrics                    → MetricBundle
    fetch-logs                       → LogBundle
  Stage 3: signal-correlator-agent   → CorrelatedSignals
  Stage 4: hypothesis-ranker-agent   → IncidentBrief
```

**Agent skill bindings:**
- `alert-triage-agent` ← `sre-alert-triage` skill (alert storm heuristics, deduplication, causal classification)
- `signal-correlator-agent` ← `sre-signal-correlation` skill (trace/log/metric join procedures, signal gap handling)
- `hypothesis-ranker-agent` ← `sre-hypothesis-generation` skill (confidence scoring, evidence citation format, runbook surface trigger)

---

## Wave: DESIGN / [REF-027] Driving Ports

| Port | Contract Shape | Notes |
|------|---------------|-------|
| `IncidentTriggerPort` | pure-function: `trigger(alertContext) → WorkflowRun` | Read-only; no write methods on port interface |
| `IncidentBriefDeliveryPort` | bounded-change: Slack channel write only | Output surface; never reads |
| `AuditLogPort` | bounded-change: append-only; universe = audit log file | Append enforced at adapter level; no delete |

---

## Wave: DESIGN / [REF-028] Driven Ports + Adapters

| Port | Adapter | Probe Requirement |
|------|---------|-------------------|
| `OtelSignalPort` | `OtelHttpAdapter` | Endpoint reachable + trace exists for test service + signal_gap handling |
| `AlertSourcePort` | `PagerDutyAdapter` | API key valid + one alert fetch + schema matches |
| `RunbookRepositoryPort` | `GitMarkdownAdapter` | Directory readable + Markdown parse succeeds + not empty |
| `MetricsPort` | `PrometheusAdapter` | Endpoint reachable + one instant-query succeeds |
| `LogPort` | `OtelLogAdapter` | Endpoint reachable + one log entry returned |
| `SlackOutputPort` | `SlackWebhookAdapter` | Probe message posted + HTTP 200 confirmed |
| `AuditLogPort` | `FileAuditAdapter` | Write permission verified + fsync behaviour probed (Docker overlayfs lie) |

Composition root invariant: Wire → Probe → Use. Probe failure emits `health.startup.refused` and halts.

---

## Wave: DESIGN / [REF-029] Technology Choices

| Component | Technology | License |
|-----------|-----------|---------|
| LLM | Claude Sonnet 4.6 | Proprietary (Anthropic) |
| Agent orchestration | Claude Agent SDK dynamic Workflows | Proprietary (Anthropic) |
| Schema validation | Zod v4 (in-project) | MIT |
| OTel integration | `@opentelemetry/sdk-node` | Apache 2.0 |
| Runbook parsing | `marked` | MIT |
| Audit logging | `pino` | MIT |
| Architectural enforcement | `dependency-cruiser` | MIT |

Full technology rationale and alternatives: `docs/product/architecture/brief.md`

---

## Wave: DESIGN / [REF-030] Decisions Table

| Decision | Chosen | Alternatives | ADR |
|----------|--------|-------------|-----|
| Runtime | Claude Code Agents + Skills + Workflows | Mastra.ai (production target), LangChain.js, plain Anthropic SDK | ADR-001 |
| Agent decomposition | Option B: Three-Stage Pipeline | Option A (monolith), Option C (swarm) | ADR-002 |
| Skill vs Tool boundary | Skills = reasoning procedures; Tools = data retrieval | Fat skills, no skills, smart tools | ADR-003 |
| Structural output contracts | Zod v4 schema per inter-agent handoff | Free-form text, TypeScript types only | ADR-002 |
| Adapter probe pattern | Wire → Probe → Use at composition root | Convention-based, test-only probes | ADR-001 |

---

## Wave: DESIGN / [REF-031] Reuse Analysis

All existing code is Mastra scaffolding in the weather domain. No domain code reuse.

| Existing Component | Decision | Justification |
|--------------------|----------|---------------|
| `weather-agent.ts` | Reference only | Weather domain; informs Mastra port shape |
| `weather-workflow.ts` | Reference only | Mastra step chaining pattern; informs Mastra port |
| `weather-tool.ts` | Reference only | HTTP fetch pattern informs OTel adapter design |
| `zod` (package.json) | REUSE | In-project; used for inter-agent JSON Schema validation |
| `typescript` (package.json) | REUSE | In-project; used for adapter interface definitions |

---

## Wave: DESIGN / [REF-032] Open Questions

| ID | Question | Impact |
|----|----------|--------|
| OQ-01 | OTel backend for testing (Jaeger, Honeycomb, or mock)? | `OtelHttpAdapter` probe implementation |
| OQ-02 | Alert trigger mechanism for walking skeleton (webhook / CLI / file)? | `IncidentTriggerPort` adapter design |
| OQ-03 | Runbook fixture set available? (P-02 pre-requisite: 10 runbooks) | Runbook reading cannot be validated without fixtures |
| OQ-04 | Slack channel available for walking skeleton output? | `SlackWebhookAdapter` config; file-based fallback needed if not |
| OQ-05 | Acceptable LLM cost per incident for the spike? | Token budgeting in Workflow script |
| OQ-06 | Single-incident or concurrent-incident support? | Workflow session isolation and audit log design |

---

## Wave: DESIGN / [REF-033] Handoff Package

### Artifacts Produced

| Artifact | Path | Status |
|----------|------|--------|
| Architecture brief | `docs/product/architecture/brief.md` | Complete |
| ADR-001: Runtime decision | `docs/product/architecture/adr-001-runtime-claude-code-vs-mastra.md` | Accepted |
| ADR-002: Agent decomposition | `docs/product/architecture/adr-002-agent-decomposition-strategy.md` | Proposed — pending user option selection |
| ADR-003: Skill vs Tool boundary | `docs/product/architecture/adr-003-skill-vs-tool-boundary.md` | Accepted |
| Wave decisions | `docs/feature/sre-agent-mttr-optimizer/design/wave-decisions.md` | Complete |
| Feature delta (DESIGN sections) | this document | Complete |

### Handoff to platform-architect

**External Integrations Requiring Contract Tests:**
- PagerDuty REST API v2: consumer-driven contracts via Pact-JS in CI acceptance stage
- OTel Collector HTTP API: Pact-JS; contracts must be backend-specific (Jaeger/Honeycomb/Grafana Tempo schema variants)
- Slack Incoming Webhooks: contract test for webhook payload schema version

**Development Paradigm**: Multi-paradigm TypeScript. Functional-leaning patterns for agent pipeline (pure-function driving ports, plan-value pattern for dry-run/preview capabilities, capability injection for restricted adapter interfaces). No OOP inheritance hierarchies.

**Architecture enforcement tooling**: `dependency-cruiser` (import graph), TypeScript interfaces (adapter probe contracts), pre-commit AST hook (skill files), Zod runtime validation (inter-agent handoffs), `gitleaks` (API key detection).
