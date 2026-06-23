# Taste Evaluation — sre-agent-mttr-optimizer

Wave: DIVERGE
Phase: 4 — Taste Evaluation
Date: 2026-06-23
Skill: nw-taste-evaluation

---

## Weight Lock — BEFORE SCORING

*Weights are declared and locked before any option is scored. No retroactive adjustment.*

**Product type**: Developer tool (SRE-grade; on-call incident context; high-stress environment).

**Weight selection rationale**:

Using the Developer Tool column from the taste-evaluation skill as baseline, with one adjustment:

| Criterion | Developer Tool Default | Applied Weight | Adjustment Rationale |
|-----------|----------------------|----------------|----------------------|
| DVF (avg) | 25% | 25% | No adjustment — developer tool baseline appropriate |
| Subtraction (T1) | 15% | 20% | Raised: at 3am under P0 stress, cognitive minimalism is life-or-death. Every additional concept or feature increases the chance of misuse. SRE tools have historically failed by over-featuring (DISCOVER: I-03 "I have three tabs open — I'm the correlation engine"). Subtraction is more important here than generic developer tool baseline suggests. |
| Concept Count (T2) | 20% | 20% | No adjustment — developer tool baseline appropriate |
| Progressive Disclosure (T3) | 15% | 10% | Lowered: incident tools are used by trained practitioners, not first-time users. Progressive disclosure matters less than it does in consumer apps. The SRE will learn the interface during setup, not at 3am. |
| Speed-as-Trust (T4) | 25% | 25% | No adjustment — Speed-as-Trust is critical for incident tools. Every second of latency in a diagnostic response directly extends MTTR. The 25% developer tool default is already elevated vs. consumer; maintain. |

**Final weights**: DVF 25% + T1 20% + T2 20% + T3 10% + T4 25% = **100%**

---

## Phase 1: DVF Filter

Scoring scale: 1–5 per lens. Elimination threshold: total DVF average < 6/15 (i.e., average < 2.0 per lens, equivalent to total < 6).

*Correct threshold per skill: DVF total < 6 → eliminated. Max total = 15 (3 lenses × 5).*

### DVF Scores

| Option | Desirability (D) | Feasibility (F) | Viability (V) | DVF Total | DVF Avg | Status |
|--------|-----------------|-----------------|---------------|-----------|---------|--------|
| OPT-1: Incident Brief | 5 | 4 | 4 | 13 | 4.33 | PASS |
| OPT-2: Continuous Scribe | 4 | 3 | 3 | 10 | 3.33 | PASS |
| OPT-3: Differential Diagnosis | 4 | 4 | 4 | 12 | 4.00 | PASS |
| OPT-4: Multi-Persona Hub | 3 | 3 | 3 | 9 | 3.00 | PASS |
| OPT-5: Runbook-First Matcher | 4 | 4 | 4 | 12 | 4.00 | PASS |
| OPT-6: Self-Hosted Silent Partner | 4 | 3 | 3 | 10 | 3.33 | PASS |

**All 6 options pass the DVF filter.** No options eliminated.

### DVF Score Rationale

**OPT-1 (Incident Brief) — D:5, F:4, V:4**
- D:5 — Matches the ghost-in-the-chair pattern exactly (8/10 DISCOVER resonance). The desired outcome is a brief delivered before the SRE asks. DISCOVER OPP-01 scored 15.5 — highest opportunity score. Maximum desirability.
- F:4 — OTel signal ingestion is mature (Mastra framework, OTel SDK). LLM reasoning over spans/metrics/logs is a solved technical problem. Core risk: LLM context window for large incident signal volumes, and reasoning quality over noisy OTel data. One point off for these open risks (DISCOVER YELLOW feasibility).
- V:4 — Clear SaaS subscription model (per-seat or per-incident). LLM inference cost per incident is a known model-able risk (DISCOVER P-04). Strong strategic differentiation (OTel-native + GIC). One point off for unvalidated pricing and LLM cost uncertainty.

**OPT-2 (Continuous Scribe) — D:4, F:3, V:3**
- D:4 — Both live investigation and post-mortem documentation are validated pain points. However, DISCOVER explicitly de-prioritized post-mortem automation (P-03 persona secondary to P-01 for MVP). Value is real but not the primary job. One point off.
- F:3 — Stateful, long-running incident process is technically harder than webhook-triggered. Requires persistent memory, incident lifecycle management, event deduplication, and a post-mortem draft generation pipeline. Two points off for architectural complexity above OPT-1.
- V:3 — Harder to price — "incident lifecycle management" is a different category from "diagnostic briefing." Competes with Incident.io directly on the incident management positioning. Two points off for unclear differentiation story and pricing ambiguity.

**OPT-3 (Differential Diagnosis) — D:4, F:4, V:4**
- D:4 — Resonates with the "I want to give 3 data points and get a ranked hypothesis" pattern (DISCOVER I-05). However, asking the SRE questions interrupts the investigation start vs. a proactive brief. 8/10 preferred "handed to me on a plate" (ghost-in-the-chair). The Socratic model is correct for low-confidence situations but may frustrate SREs in high-confidence ones. One point off.
- F:4 — Technically simpler than OPT-1 in one dimension (LLM reasoning over fewer signals + SRE input = higher quality with less inference) but requires dialogue state management. Net equivalent to OPT-1.
- V:4 — Clear product positioning as "AI incident interviewer." Unique in the market. Pricing equivalent to OPT-1. One point off vs. theoretical max due to interaction model risk (SRE may find the Q&A adds friction at 3am).

**OPT-4 (Multi-Persona Hub) — D:3, F:3, V:3**
- D:3 — The Incident Commander job is real (DISCOVER P-02) but was not the primary validated persona. The hypothesis that "SREs and Commanders co-inhabit the same tool" requires validation. The primary focus should be solving the on-call SRE job first. Two points off for persona dilution and unvalidated multi-role assumption.
- F:3 — Multi-format rendering is technically achievable but adds complexity over a single-persona brief. Testing usability for three different audiences simultaneously is harder. Two points off.
- V:3 — Unclear whether a multi-persona product would be priced as incident management (per-incident) or per-seat (which personas?). Harder to sell in the SRE tooling category. Two points off.

**OPT-5 (Runbook-First Matcher) — D:4, F:4, V:4**
- D:4 — Senior SRE pattern-recognition resonance is real (DISCOVER I-05: "A senior would recognize it in five minutes"). However, DISCOVER also established that runbook currency is 30-70% stale (C-03). The primary constraint is that runbooks cannot be treated as ground truth. Semantic matching against stale runbooks produces a stale hypothesis. One point off for this structural risk.
- F:4 — Retrieval-augmented generation is a mature, well-understood pattern. Simpler to implement than full OTel signal reasoning. The OTel signal fallback for novel incidents adds complexity but is separable. One point off for the hybrid architecture requiring both retrieval and reasoning paths.
- V:4 — Runbook-as-knowledge-base is a clear product angle. Can be positioned as "institutionalized institutional knowledge." One point off for market positioning ambiguity (is this a runbook tool or an incident tool?).

**OPT-6 (Self-Hosted Silent Partner) — D:4, F:3, V:3**
- D:4 — Strong product-market fit for regulated industries (healthcare, fintech, government) where data residency is non-negotiable. DISCOVER corpus included I-09 (healthcare SaaS) and I-01 (payments platform) — real evidence of this segment. One point off because CLI-first query model is less proactive than the ghost-in-the-chair brief pattern (requires SRE to initiate query).
- F:3 — Continuous pre-computation architecture running as an OTel Collector sidecar requires significant infrastructure design: what compute resources, what persistence model, how does it scale during a multi-hour P0 with high signal volume? Two points off for open architectural questions.
- V:3 — Self-hosted model means no SaaS revenue stream without significant enterprise sales motion. Monetization paths (open-source + enterprise tier, or fully commercial self-hosted license) are harder to validate. Two points off.

---

## Phase 2: Taste Criteria Scoring

All 6 options pass DVF. All 6 are scored on 4 taste criteria.

### T1: Subtraction

*Test: Could this option achieve its goal with one fewer feature/concept/step?*

| Option | Score | Rationale |
|--------|-------|-----------|
| OPT-1: Incident Brief | 5 | Core value = one artifact delivered to one channel with zero user action. Nothing can be removed without breaking the value. The 3-part structure (what/why/next) is the minimum necessary format for an actionable brief. |
| OPT-2: Continuous Scribe | 3 | Two artifacts (live brief + post-mortem draft) where one would suffice for the primary job. The post-mortem output is additive value, not core value. The stateful process model adds complexity for the primary use case (live investigation). Multiple removable elements. |
| OPT-3: Differential Diagnosis | 4 | Three targeted questions + hypothesis synthesis. Could it work with two questions? Possibly. However, the three-question structure is grounded in the mechanism (one yes/no context question, one deployment question, one scope question). One minor element could be reduced; core intact. |
| OPT-4: Multi-Persona Hub | 2 | Three distinct output formats where one (the SRE Technical Brief) would deliver the primary job. The Commander Brief and Leadership Summary are additive features that could be removed without breaking the core value. Clearly has non-essential parts — the multi-persona rendering is a feature layer, not core. |
| OPT-5: Runbook-First Matcher | 4 | Retrieval-first with OTel fallback is the minimum necessary architecture for the stated goal (fast response for known patterns, coverage for novel ones). The hybrid architecture is justified by the distinct speed profiles. One minor optimization possible (threshold tuning), but core structure is minimal. |
| OPT-6: Self-Hosted Silent Partner | 4 | CLI interface + continuous pre-compute is minimal for the on-prem constraint. The Docker sidecar deployment model is the simplest self-hosted deployment. Could it be even simpler (just a binary)? Yes — minor element removable; core intact. |

### T2: Concept Count

*Test: How many new mental concepts does a first-time user need to learn?*

| Option | Score | Rationale |
|--------|-------|-----------|
| OPT-1: Incident Brief | 5 | Zero new concepts. "Brief" maps to existing mental model (incident briefing on a bridge call, doctor's notes). The three-part structure (what/why/next) is universal narrative. The on-call SRE receives a Slack message — they already know Slack. Nothing new to learn. |
| OPT-2: Continuous Scribe | 3 | Two new concepts: (1) "agent that lives through the whole incident" (new — existing tools are point-in-time), (2) "auto-drafted post-mortem" (familiar goal, unfamiliar mechanism). Two concepts, introduced sequentially through incident lifecycle. |
| OPT-3: Differential Diagnosis | 3 | Two new concepts: (1) "AI diagnostic interview" (new interaction pattern), (2) "explicit confidence percentage on hypothesis" (familiar from weather forecasts; unfamiliar in incident context). The Q&A interaction model is the primary learning curve. |
| OPT-4: Multi-Persona Hub | 2 | Three or more concepts: (1) "incident hub," (2) persona-differentiated outputs (SRE Brief / Commander Brief / Leadership Summary), (3) collapsible multi-section format in Slack, (4) multi-role usage model. Some concepts are interdependent (you must understand persona segmentation to understand why the same incident has three different sections). |
| OPT-5: Runbook-First Matcher | 4 | One new concept: "runbook semantic matching." Maps to a familiar mental model (senior SRE pattern recognition: "this looks like Incident #342"). The confidence score is intuitive (percentage). The fallback to signal analysis is invisible to the user — they don't need to understand the hybrid architecture. |
| OPT-6: Self-Hosted Silent Partner | 3 | Two new concepts: (1) "sidecar agent that monitors continuously without announcing itself," (2) "natural language CLI for incident queries." CLI is familiar to SREs; the NL interface on top of CLI is one new concept. The sidecar deployment model requires ops understanding (new concept for SREs who don't run infrastructure). |

### T3: Progressive Disclosure

*Test: Does the first interaction expose only what's needed for the first use case?*

| Option | Score | Rationale |
|--------|-------|-----------|
| OPT-1: Incident Brief | 5 | First interaction = receive a Slack message. One action: read the brief. No onboarding required at incident time. Setup complexity is pre-incident (configuring the webhook, OTel connection, runbook path). At the moment of maximum stress, the SRE sees one thing: the brief. Depth (full trace context, runbook steps) available via links. |
| OPT-2: Continuous Scribe | 3 | First interaction during incident = receive diagnostic brief (same as OPT-1). Second interaction = see live timeline updates as the incident progresses. Third interaction = receive post-mortem draft after resolution. The three stages are sequential and well-scoped. Sequencing is logical, but the SRE must develop a mental model of "what is the agent doing right now?" across three phases. |
| OPT-3: Differential Diagnosis | 3 | First interaction = receive 3 questions to answer. The SRE must respond before receiving diagnostic value. This is an inversion of OPT-1's progressive disclosure — the first touch requires action, not just consumption. Sequencing is logical (questions → hypothesis is a clear cause-effect), but the immediate demand for input before value delivery is a disclosure friction. 2–3 features exposed at first touch. |
| OPT-4: Multi-Persona Hub | 2 | First interaction = receive a Slack message with 3 collapsible sections. The SRE must choose which section to read — immediately exposing multiple paths. This is not progressive; it is simultaneous. The on-call SRE at 3am should not have to choose between "SRE Brief," "Commander Brief," and "Leadership Summary." Choice at the first touch is a design failure in this context. |
| OPT-5: Runbook-First Matcher | 4 | First interaction = receive a single recommendation: "This matches Incident #342 — verify rate limit config on redis-gateway. Confidence 78%." One action, one recommendation. The confidence score + "next" action is all that's exposed first. Links to full hypothesis reasoning, fallback signal analysis, and alternative hypotheses are secondary. Nearly as minimal as OPT-1. |
| OPT-6: Self-Hosted Silent Partner | 4 | First interaction during incident = type `sre ask "why is checkout slow?"`. One command, one response. Depth available via follow-up queries. The CLI interaction model is well-understood by SREs; no new navigation required. Minor friction: SRE must initiate the query (vs. OPT-1 which delivers unprompted). |

### T4: Speed-as-Trust

*Test: Does this option introduce latency, friction, or steps that erode the sense of speed?*

| Option | Score | Rationale |
|--------|-------|-----------|
| OPT-1: Incident Brief | 4 | Delivery target: <60 seconds from alert fire. This is a fixed latency imposed by OTel query + LLM reasoning. However, the SRE has nothing to do during those 60 seconds — they're reading the alert, silencing their phone, opening Slack. The brief arrives as they orient themselves. The latency is masked by natural cognitive transition time. Minor: during that 60 seconds, the SRE has no progress feedback unless a "brief incoming" status message is sent immediately on alert. One point off for the 60-second hard latency vs. instant feedback. |
| OPT-2: Continuous Scribe | 3 | Same 60-second initial brief latency as OPT-1. Additional: the post-mortem draft has a processing delay after incident close (must reconstruct the event log into a structured timeline). Noticeable latency at two points; payoff is clear for both. |
| OPT-3: Differential Diagnosis | 3 | Total time-to-hypothesis = 60s (signal analysis) + time SRE takes to answer 3 questions + roundtrip LLM processing. Realistically: 3-5 minutes total. This is a meaningful increase over OPT-1's 60 seconds, even though the hypothesis quality may be higher. The wait points include both system processing and human response time, creating compounded latency. |
| OPT-4: Multi-Persona Hub | 3 | Same brief latency as OPT-1, plus additional time to render 3 different output formats. Minor processing overhead, but more importantly: the Incident Commander and SRE both experience the same initial wait, which amplifies pressure during a P0 bridge call. No individual wait point is long; the cumulative effect under stress is the issue. |
| OPT-5: Runbook-First Matcher | 5 | For known patterns: <5 seconds (retrieval, not reasoning). This is the fastest hypothesis delivery of any option. The SRE receives a match before they finish reading the alert. For novel incidents (fallback to OTel analysis): 60 seconds — same as OPT-1. The bimodal speed profile rewards the most common case (recurring incidents) with maximum speed, penalizes only novel incidents with standard latency. |
| OPT-6: Self-Hosted Silent Partner | 5 | Pre-computed continuous context means <5 second response to any CLI query. The pre-computation eliminates the wait point entirely — context is always ready. The SRE waits only for their own query transmission and LLM formatting. Maximum speed-as-trust: the system communicates "I already know the answer" through its response latency. |

---

## Phase 3: Weighted Scoring Matrix

**Weight reminder**: DVF 25%, T1 20%, T2 20%, T3 10%, T4 25%

**DVF contribution**: Using DVF average (out of 5) as the DVF score for weighting purposes. [DVF Total / 3 = DVF Avg out of 5]

| Option | DVF Avg | T1 Sub | T2 Concept | T3 Prog | T4 Speed | Weighted Total |
|--------|---------|--------|------------|---------|----------|----------------|
| OPT-1: Incident Brief | 4.33 | 5 | 5 | 5 | 4 | **4.63** |
| OPT-2: Continuous Scribe | 3.33 | 3 | 3 | 3 | 3 | **3.17** |
| OPT-3: Differential Diagnosis | 4.00 | 4 | 3 | 3 | 3 | **3.55** |
| OPT-4: Multi-Persona Hub | 3.00 | 2 | 2 | 2 | 3 | **2.55** |
| OPT-5: Runbook-First Matcher | 4.00 | 4 | 4 | 4 | 5 | **4.15** |
| OPT-6: Self-Hosted Silent Partner | 3.33 | 4 | 3 | 4 | 5 | **3.82** |

**Calculation verification**:
- OPT-1: (4.33 × 0.25) + (5 × 0.20) + (5 × 0.20) + (5 × 0.10) + (4 × 0.25) = 1.083 + 1.00 + 1.00 + 0.50 + 1.00 = **4.58** *(revised below)*
- OPT-5: (4.00 × 0.25) + (4 × 0.20) + (4 × 0.20) + (4 × 0.10) + (5 × 0.25) = 1.00 + 0.80 + 0.80 + 0.40 + 1.25 = **4.25**
- OPT-6: (3.33 × 0.25) + (4 × 0.20) + (3 × 0.20) + (4 × 0.10) + (5 × 0.25) = 0.833 + 0.80 + 0.60 + 0.40 + 1.25 = **3.88**
- OPT-3: (4.00 × 0.25) + (4 × 0.20) + (3 × 0.20) + (3 × 0.10) + (3 × 0.25) = 1.00 + 0.80 + 0.60 + 0.30 + 0.75 = **3.45**
- OPT-2: (3.33 × 0.25) + (3 × 0.20) + (3 × 0.20) + (3 × 0.10) + (3 × 0.25) = 0.833 + 0.60 + 0.60 + 0.30 + 0.75 = **3.08**
- OPT-4: (3.00 × 0.25) + (2 × 0.20) + (2 × 0.20) + (2 × 0.10) + (3 × 0.25) = 0.75 + 0.40 + 0.40 + 0.20 + 0.75 = **2.50**

**Final Verified Scores:**

| Option | DVF Avg | T1 Sub | T2 Concept | T3 Prog | T4 Speed | Weighted Score |
|--------|---------|--------|------------|---------|----------|----------------|
| OPT-1: Incident Brief | 4.33 | 5 | 5 | 5 | 4 | **4.58** |
| OPT-5: Runbook-First Matcher | 4.00 | 4 | 4 | 4 | 5 | **4.25** |
| OPT-6: Self-Hosted Silent Partner | 3.33 | 4 | 3 | 4 | 5 | **3.88** |
| OPT-3: Differential Diagnosis | 4.00 | 4 | 3 | 3 | 3 | **3.45** |
| OPT-2: Continuous Scribe | 3.33 | 3 | 3 | 3 | 3 | **3.08** |
| OPT-4: Multi-Persona Hub | 3.00 | 2 | 2 | 2 | 3 | **2.50** |

---

## Ranking

1. OPT-1: The Incident Brief — **4.58**
2. OPT-5: Runbook-First Hypothesis Matcher — **4.25**
3. OPT-6: Self-Hosted Silent Partner — **3.88**
4. OPT-3: Differential Diagnosis Protocol — **3.45**
5. OPT-2: Continuous Incident Scribe — **3.08**
6. OPT-4: Multi-Persona Incident Hub — **2.50**

---

## Criterion-Level Breakdown

### Why OPT-1 leads

OPT-1 achieves the maximum possible score on three of five criteria (T1 Subtraction: 5, T2 Concept Count: 5, T3 Progressive Disclosure: 5). It scores 4.33 on DVF — the highest of any option. The only criterion where it loses ground is T4 Speed (4 vs. 5 for OPT-5 and OPT-6), because the 60-second LLM reasoning pass introduces an irreducible latency period that the pre-computed options avoid. That 60-second wait is real but masked by natural cognitive transition time. The T4 difference is real, not illusory — but it is outweighed by the superior scores on all other criteria.

### Why OPT-5 is the strong second

OPT-5 achieves the maximum T4 score (5) and scores well across all other criteria. Its key advantage is speed for the most common case: known incident patterns get a <5 second hypothesis, which is faster than any other option. Its primary risks — runbook staleness (C-03) and the assumption that ≥50% of incidents are recurring — are well-documented constraints, not surprises. If those risks prove lower than feared (i.e., if runbook quality at target customers is higher than the DISCOVER average of 30-40% stale), OPT-5's case strengthens significantly.

### Why OPT-6 is the interesting third

OPT-6 achieves maximum T4 (5) through pre-computation and scores well on Subtraction (4) and Progressive Disclosure (4). It loses ground on DVF (3.33 — lowest viability due to SaaS revenue model uncertainty) and Concept Count (3 — the sidecar deployment model requires operational understanding). Its strategic insight is unique: it is the only option in the set that addresses the regulated-industry customer segment (healthcare, fintech, government) that cannot send telemetry data to a third-party SaaS. This is a market segment not served by any option above it in the ranking.

### Why OPT-4 scores last

OPT-4 performs poorly on all taste criteria: Subtraction (2 — clearly has non-essential features), Concept Count (2 — three or more new concepts introduced simultaneously), and Progressive Disclosure (2 — choice between three personas at first touch is a design failure in a high-stress context). The core idea (multi-persona output) is architecturally interesting but violates the principles of subtraction and cognitive minimalism that matter most for a product used at 3am under P0 pressure.

---

## Gate G4 Validation

| Criterion | Status | Evidence |
|-----------|--------|----------|
| DVF filter applied to all 6 options | PASS | All 6 options scored; all 6 pass threshold |
| Weights documented and locked before scoring | PASS | Weight table with rationale appears before any score |
| All surviving options scored on all 4 taste criteria | PASS | All 6 options scored on DVF, T1, T2, T3, T4 |
| Weighted ranking complete | PASS | Scores range 2.50–4.58 with clear ordering |
| Recommendation traceable to scores | PASS — see recommendation.md | Top score = recommended option |
| Dissenting case included | PASS — see recommendation.md | Second-place option dissent documented |
| Decision statement for DISCUSS wave explicit | PASS — see recommendation.md | Decision statement ends recommendation.md |

**G4: PASS. Proceed to peer review.**
