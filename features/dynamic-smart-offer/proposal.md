# Dynamic Smart Offer — Proposal

**Jira:** RDESQ-1900
**Target Launch:** Q1 pilot by March 31, 2026
**Authors:** Eliza Wall & Alicia James (PM/Product)
**Team:** Expert Workspace — Expert Product: Expert Workspace | Sell
**Estimated dev:** 11 weeks / 1 engineer
**Repos involved:** expert-workspace, expert-workspace-api, expert-workspace-config

---

## Why

The current "Smart Offer" is static and does not leverage existing customer context:

- Messaging is generic and misaligned to customer needs
- Known objections are not proactively addressed
- Tone is often transactional/tone-deaf

We are underutilizing customer signals that could materially improve offer conversion.

**Evidence:**

- Field "Smart Pitch" pilot (DiD analysis): Test +1.60 vs Control +0.06 (SP100); 80% confidence
- Alpha "Personalized Offer": Positive for Cricket, negative for Verizon; AI-generated copy underperformed pre-approved marketing language → MVP grounds outputs in pre-approved pitch pool
- Low click-through rate (~5%) in field pilot limits signal strength

---

## What

Evolve the existing Smart Offer into a context-aware system that dynamically generates personalized offer guidance based on real-time transcript signals.

**DS Inputs (MVP — real-time transcript only):**
- Tone / emotional cues from transcript
- Customer attributes inferred from transcript (tech savviness, children, devices)

**DS Outputs:**
1. **Tone Guidance** — concise recommendation on how the expert should frame the offer
2. **Three Pitch Points** — selected from ~20 pre-approved, marketing-written pitch points, ranked dynamically; light contextual tailoring only (no net-new generation)

**UI Changes:**
1. Tone Guidance section injected above offer scripting
2. Three Pitch Points replace the current "Core Benefits" section
3. "Example of Coverage" section hidden (intent fulfilled by personalized pitch points)
4. Manual "Show latest" button — only visible when newer data is available in the store
5. Static fallback when DS output unavailable (existing three static benefits shown; placeholder for tone guidance)

**Static sections unchanged:** Transition, discovery, engagement, price, ask for sale, ask to send link, all detection models.

---

## MVP Exclusions (Post-MVP Roadmap)

- Historical call transcripts
- Claims data (device type, peril, claim status)
- C360 / prior call history
- Callback awareness
- Objection-aware guidance
- Rebuttal/objection handling triggered live during call
- Dynamic objection handling
- Personalized transition/discovery/engagement questions
- Logic to determine most likely product to purchase

---

## Success Metrics

**Primary:** Offer usage (adoption rate), NSP100
**Secondary:** NPS, Compliance rate

**Measuring usage of generated content:**
- DS: Retroactively compare full transcript to generated content (~400–500 call batch)
- Quality team: Targeted manual review
- Future: Real-time detection models for expert mention of generated content

---

## Constraints & Risks

- AI-generated copy underperforms pre-approved marketing language (Alpha finding) — mitigated by grounding in approved pool
- Legal/compliance risk from dynamically generating new benefit copy — avoided in MVP
- Prior call transcripts may be incomplete, delayed, or unavailable — MVP sidesteps this (real-time only)
- GAIA event delivery reliability — resilience strategy TBD
- Decline reasons not consistently captured across channels
