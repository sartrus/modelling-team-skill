# DraftPlan — Validation Strategy Analysis
2026-03-25 (from NextIdea_generator session)

## Context
Cross-referenced DraftPlan against the NextIdea_generator scoring framework and idea-refiner skill. Fetched getdraftplan.com live. Ran full adversarial debate (Advocate + Critic, 2 rounds).

---

## Idea Refiner Verdict: CONDITIONAL GO

Build and launch only if organic acquisition works before spending on paid traffic. Pain is genuine (5/5), WTP is structurally anchored against $500–$3,000 professional alternative, 3-agent validation architecture solves a real gap. Two structural risks survived:

1. **Distribution has no flywheel** — organic-only, episodic use (one model per funding round), keyword cluster dominated by content farms. No repeat purchase cadence.
2. **Formatting moat expires 6–12 months** — Claude Artifacts / Cursor closing the gap between raw LLM output and formatted Excel. Durable moat must be domain-specific validation accuracy, not the file format.

Full report: `NextIdea_generator/idea_reports/91_ai_financial_model_generator.md`

---

## $200 Paid Test — Is It the Right Move?

**Yes, but the current landing page tests the wrong thing.**

Current CTA: "Join the Waitlist — First Model Free" → measures interest in a free product, not WTP.

**Required change before running ads:**
- Add a payment-intent CTA: "Get My Model — $29" → Stripe/Gumroad (fulfill manually for now)
- Keep waitlist as secondary CTA
- Remove "First Model Free" from waitlist button (trains visitors to expect zero price)
- Sharpen subheadline to one ICP (startup founders or SMB, not both)
- Move comparison table earlier in scroll (strongest trust element)

**Run two ad variations to test keyword intent:**
1. Template-intent: "startup financial model template", "excel financial model startup"
2. AI-intent: "ai financial model generator", "financial projections generator"

If template-intent CTR >> AI-intent → position as template replacement, not AI tool.

**Phase 1 success criteria (from existing memory):** CTR >2%, landing conversion >3%, cost/lead <$20.
**Updated success criteria for payment-intent test:** at least 1 payment attempt per $50 spent.

---

## Subscription Model — Does It Work?

**No for the current ICP (founders), yes for a different ICP.**

Founders: episodic use (1 model/funding round) → cancel after month 1.

Service provider ICP where subscription works: fractional CFOs, financial consultants building 3–10 models/month for clients. BUT — accountants don't build forward-looking models (they work backward from history). The actual TAM for "financial service providers who build models repeatedly" is fractional CFOs specifically — small, hard-to-reach, and they have their own templates as their craft identity.

**Recommendation:** Stay with pay-per-model for Phase 1. Revisit subscription only after validating ICP more precisely.

---

## Manual "Wizard of Oz" Test (parallel to paid test)

Before building the automated 3-agent pipeline, run 3–5 manual models:
- Post in r/startups: "Building a financial model generator — offering 3 free models in exchange for feedback"
- Build manually using Claude + Excel (3–4 hours total)
- Validates: output quality acceptable? Would they have paid? What did wizard miss?

This costs $0 and answers questions that $200 in ads cannot.

**Recommended order:**
1. Fix landing page CTA (this week)
2. Run 3 manual models + $200 paid test simultaneously
3. If 3+ payment attempts + manual quality passes → build MVP
