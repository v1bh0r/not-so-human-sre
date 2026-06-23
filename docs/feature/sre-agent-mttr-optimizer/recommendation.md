# Recommendation — sre-agent-mttr-optimizer

Wave: DIVERGE
Date: 2026-06-23
Handoff to: product-owner (DISCUSS wave)

---

## Top 3 Options

### Option 1: The Incident Brief — Score 4.58

**Why it scores well**: Achieves the maximum score on three of five taste criteria. T1 Subtraction (5/5): one artifact, one channel, zero user action — nothing can be removed without breaking the value. T2 Concept Count (5/5): zero new mental models required — a "brief" maps to existing cognitive constructs (briefing notes, doctor's assessment). T3 Progressive Disclosure (5/5): first interaction is reading a Slack message — no setup required at incident time, maximum depth available via links. Highest DVF average (4.33) — the strongest combination of desirability, feasibility, and viability in the option set.

**Core trade-off**: Speed vs. proactiveness. OPT-1 introduces a 60-second irreducible latency (OTel query + LLM reasoning). Competing options (OPT-5, OPT-6) can deliver hypotheses in <5 seconds through retrieval or pre-computation. That 60-second wait is real — but it is masked by natural cognitive transition time (alert → silence phone → open Slack → orient). The tradeoff is accepted because proactive push (no action required from SRE) beats reactive pull (SRE must query) for the primary job, and 60 seconds is materially better than 10–30 minutes.

**Key risk**: LLM reasoning quality over OTel signals at production scale. A hypothesis that is wrong with high apparent confidence will destroy trust faster than no hypothesis at all (DISCOVER E-03: "Feel protected from blame if something goes wrong when following AI recommendations"). Mitigation: calibrated confidence scores, signal citations that let the SRE verify the reasoning (C-04), and explicit "signal gap" output state when OTel coverage is incomplete (C-02).

**Hire criteria**: Choose OPT-1 when: (a) the SRE team's primary pain is the first 10–30 minutes of manual correlation before any hypothesis, (b) the team is on a mixed OTel-native stack (Prometheus + Loki + Tempo) and cannot adopt a single-vendor observability platform, (c) trust in AI-generated hypotheses is currently low and must be built through transparent evidence (the brief format is designed for explainability), and (d) SaaS deployment is acceptable.

---

### Option 2: Runbook-First Hypothesis Matcher — Score 4.25

**Why it scores well**: Maximum T4 Speed-as-Trust score (5/5) — <5 second hypothesis delivery for known incident patterns. This is the fastest path from alert to hypothesis of any option in the set. Strong Subtraction (4/5) and Concept Count (4/5): "this looks like Incident #342" is a concept every senior SRE already uses mentally — the product externalizes and accelerates a known cognitive pattern. The fallback to OTel signal analysis for novel incidents means coverage is not sacrificed for the speed advantage.

**Core trade-off**: Known patterns vs. novel incidents. Runbook-First Matcher is dramatically faster for recurring incidents (which are the majority, per DISCOVER I-05) but relies on institutional knowledge quality for that speed advantage. If the runbook corpus is stale (DISCOVER: 30–70% stale across the interviewee corpus), the initial hypothesis for a known-pattern incident may itself be stale. C-03 (runbooks cannot be treated as executable ground truth) applies here — but the hypothesis derived from runbook matching is a starting point for investigation, not an execution instruction.

**Key risk**: The core assumption is that ≥50% of P1 incidents are variants of previously seen failure patterns. This is not validated at the individual customer level — it is inferred from the "senior SRE pattern recognition" finding (DISCOVER I-05). If a target customer is running a greenfield system with no incident history, OPT-5 loses its primary speed advantage and becomes equivalent to a more complex version of OPT-1.

**Hire criteria**: Choose OPT-5 when: (a) the team has an existing post-mortem library (even partial), (b) their primary incident types are recurring failure patterns (payment timeout, database connection exhaustion, certificate expiry), (c) speed is the single most important criterion over comprehensiveness, and (d) the team accepts that novel incidents will have a 60-second response time instead of <5 seconds.

---

### Option 3: Self-Hosted Silent Partner — Score 3.88

**Why it scores well**: Maximum T4 Speed-as-Trust (5/5) through pre-computation — the system always has the answer ready, communicating competence through response latency alone. Strong Subtraction (4/5) and Progressive Disclosure (4/5): CLI query → response is the minimal interaction model for SRE-native tooling. Strategic differentiation: the only option in the set that addresses regulated-industry customers (healthcare, fintech, government) with hard data residency requirements.

**Core trade-off**: Speed and data residency vs. proactiveness and simplicity. OPT-6 requires the SRE to initiate a query — it does not push a brief proactively. This breaks the ghost-in-the-chair pattern (AI investigates before SRE asks). The deployment model (self-hosted Docker sidecar) also requires operational investment that SaaS-first options avoid. The DVF average is the lowest of the top 3 (3.33) primarily due to viability uncertainty around self-hosted monetization.

**Key risk**: The continuous pre-computation architecture running alongside the OTel Collector must be designed for the full incident signal volume — during a P0, OTel signal throughput can spike 10–100x. If the pre-computation layer cannot keep pace with signal volume during the exact moment it matters most, the <5 second promise breaks.

**Hire criteria**: Choose OPT-6 when: (a) data residency is a hard constraint (telemetry cannot leave the customer's infrastructure perimeter), (b) the team is CLI-native and prefers query-on-demand over push notifications, (c) the customer cannot or will not adopt a SaaS product for incident investigation, and (d) the team has operational capacity to deploy and maintain a sidecar alongside their OTel Collector.

---

## Recommended Direction

**OPT-1: The Incident Brief** is the recommended design direction.

OPT-1 scores highest on the scoring matrix (4.58 vs. 4.25 for second place) because it is the most direct resolution of the primary job: eliminate the 10–30 minutes of manual signal correlation before the SRE's first hypothesis, without requiring the SRE to take any action. The ghost-in-the-chair pattern — the highest-resonance framing from DISCOVER (8/10 SREs) — is embodied most directly by an autonomous agent that delivers a brief before the SRE asks for it. OPT-1 is the design direction that most literally implements that framing.

The score advantage on subtraction and concept count is not incidental. For a product used at 3am under P0 pressure, cognitive simplicity is the core reliability requirement. The brief is a Slack message. The SRE reads it. That is the entire interaction. No new tools, no new commands, no choices. The depth — full trace context, alternative hypotheses, runbook step links — is available via embedded links that the SRE accesses when they want more, not as the primary interface. This progressive disclosure architecture is why OPT-1 achieves 5/5 on T3 and OPT-5 achieves only 4/5.

The 60-second latency vs. OPT-5's <5-second speed advantage does not change the recommendation. The DISCOVER data shows the pain is 10–30 minutes; the difference between 60 seconds and 5 seconds is noise against that baseline. More importantly, OPT-1's proactive push model means the SRE receives the hypothesis at the moment they open their incident channel — not after they remember to type a query. The behavioral friction of OPT-5's pull model (SRE must initiate) is higher than OPT-1's latency disadvantage.

**The recommendation is OPT-1 with OPT-5's retrieval layer as an architectural accelerator.** The two options are not mutually exclusive at the implementation level: OPT-1's agent can use OPT-5's retrieval-first mechanism internally — check for a runbook/post-mortem match first (fast path, <5s if found), and fall back to full OTel signal reasoning (60s path) if no match. This hybrid architecture delivers the best of both: OPT-1's proactive push UX and ghost-in-the-chair interaction model, with OPT-5's speed profile for recurring incidents. The decision for the DISCUSS wave is about the primary design direction (OPT-1's UX and interaction model), not about precluding the retrieval layer as an implementation optimization.

---

## Dissenting Case

**The case for OPT-5 (Runbook-First Hypothesis Matcher) as the primary direction:**

If DISCOVER's most relevant finding is that "senior SREs recognize recurring incidents in five minutes and junior SREs take fifteen" (I-05), then the highest-leverage product is not one that eliminates manual correlation for all incidents — it is one that transfers senior SRE pattern recognition to every SRE. OPT-5 does this more directly than OPT-1: "this looks like Incident #342" is the exact cognition a senior SRE performs, externalized as a product.

OPT-5 also has a structural advantage that the scoring matrix does not fully capture: it builds a flywheel. Every resolved incident becomes a new data point in the matching library. The product gets better with every P1 — the hypothesis matcher improves as institutional knowledge accumulates. OPT-1's LLM reasoning over signals does not automatically improve with incident history (unless fine-tuning is applied).

The case against OPT-1 as primary: the 60-second LLM reasoning pass is a fixed cost that cannot be amortized across incidents. OPT-5's <5-second retrieval amortizes all past incidents into faster future responses.

**Why this dissenting case does not change the recommendation**: The flywheel argument for OPT-5 is compelling but requires a minimum viable knowledge base before the product delivers value. A customer with zero post-mortem history and a freshly deployed Mastra agent has an empty matching library — OPT-5 falls back to OTel signal reasoning for every incident until the library builds. OPT-1 delivers full value on day one, before any incident history exists. For a new product entering a market, day-one value is more important than 12-month flywheel value. The recommended path (OPT-1 as primary UX with retrieval layer as internal optimization) eventually delivers OPT-5's flywheel as part of OPT-1's architecture — making the dissent moot by implementation.

---

## Decision for DISCUSS Wave

**Proceed with OPT-1 (The Incident Brief) as the primary design direction**, assuming:

1. LLM reasoning quality over OTel signals is sufficient to produce hypotheses that are right more often than an experienced SRE acting alone in the same time window (primary feasibility assumption — requires P-01 technical spike).
2. A 60-second delivery target from alert fire to Slack brief is achievable with Mastra + OpenAI or Anthropic APIs under typical OTel signal volumes (must be validated in technical spike).
3. The brief format (3-part: what is broken / ranked hypotheses with signal citations / recommended next action) is comprehensible under high-stress incident conditions (requires P-03 prototype usability test with 5 SREs).
4. SaaS deployment is acceptable to the initial customer segment — regulated-industry customers requiring on-premises data residency are a V2 priority (addressed by OPT-6 as a later deployment model).

**Architecture note for product-owner**: The DISCUSS wave should incorporate OPT-5's retrieval-first mechanism as an internal implementation optimization (not a separate product experience). The brief should attempt runbook/post-mortem semantic matching first; report match confidence as part of the hypothesis citation ("this hypothesis matches Incident #342 — 78% similarity") when a match is found; fall back to full OTel signal reasoning when no match clears the confidence threshold. The user experience is OPT-1's push-and-read model throughout. The speed and accuracy benefits of OPT-5 are captured internally.

**What the DISCUSS wave decides**: (1) The specific format of the incident brief (3-part prose vs. structured fields vs. hybrid), (2) the interaction model for SRE follow-up questions on the brief, (3) the trust-building roadmap from read-only investigation to opt-in autonomous actions (aligned with C-01), and (4) the integration surface between the brief and the runbook corpus (C-05: Markdown-in-Git primary format).
