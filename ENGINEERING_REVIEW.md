# Engineering Review: What Was Fixed

**Date:** January 2026
**Reviewer:** Self-critique prior to public release
**Status:** Ready for peer review

---

## Original Issues Identified

### 1. **Overclaiming Without Statistical Rigor**
- ❌ "Production-Ready System"
- ❌ "100% citation compliance" (from n=15)
- ❌ "Proven Results"
- ❌ "Transform LLMs from..."

### 2. **Missing Critical Sections**
- ❌ No Related Work section
- ❌ Insufficient Limitations discussion
- ❌ No threats to validity
- ❌ No acknowledgment of prompt engineering confound
- ❌ No ground truth verification discussion

### 3. **Marketing Tone**
- ❌ Excessive emojis in headers
- ❌ "Built with ❤️"
- ❌ Definitive claims without hedging
- ❌ No invitation for critique

### 4. **Unresolved Technical Issues**
- ❌ Fee discrepancy ($60 vs $65) glossed over
- ❌ Single-model evaluation presented as general result
- ❌ No ablation study mentioned as limitation
- ❌ No discussion of alternative approaches (RAG, fine-tuning)

---

## Fixes Applied

### Whitepaper (`docs/whitepaper.md`)

#### **Title & Status:**
```diff
- Title: "...for Reducing AI Hallucination..."
- Status: Production-Ready System
- Version: 1.0

+ Title: "Structured Evidence Enforcement for LLM Outputs..."
+ Status: Experimental / Seeking Feedback
+ Version: 0.1 (Preliminary Work)
```

#### **Abstract:**
```diff
- "CogniMaps reduce hallucination risk"
- "Key Results: 100% citation compliance"

+ "CogniMaps attempt to reduce hallucination risk"
+ "Initial Observations (n=15 questions, Gemini 2.0 Flash):"
+ "This is early work with significant limitations"
```

#### **Added Section 2: Related Work**
- RAG (Lewis et al., 2020)
- Structured output & function calling (OpenAI, Anthropic)
- Constitutional AI (Bai et al., 2022)
- Factuality work (Min et al., 2023; Gao et al., 2023)
- Legal tech AI
- **Explicit positioning:** "Incremental contribution building on structured output work"

#### **Expanded Section 9: Limitations (8 detailed subsections)**

**9.1 Small Sample Size:**
- n=15 insufficient for statistical claims
- No significance testing, no confidence intervals
- Single-model evaluation

**9.2 Prompt Engineering Confound:**
- Complex prompt may explain improvement (not validation gates)
- No ablation study performed
- Need component isolation

**9.3 Ground Truth Uncertainty:**
- **Critical:** Fee discrepancy ($60 vs $65) shows we lack verified ground truth
- "LIVE_FETCH_VERIFIED" ≠ verified ground truth
- Web pages can be outdated, cached, or misinterpreted

**9.4 Single Domain:**
- Only US patent law evaluated
- Claims about other domains are hypothetical

**9.5 Live Fetch Dependency:**
- Not all platforms support web access
- High maintenance burden

**9.6 User Experience:**
- Verbosity concerns
- Need user studies

**9.7 No Adversarial Testing:**
- Gates may have blind spots
- Regex detection can be evaded

**9.8 Lack of Baseline Comparison:**
- No comparison to RAG, fine-tuning, or other techniques

**9.9 Future Work:**
- Concrete milestones: n=100+, ablation study, multi-model, multi-domain

#### **Revised Conclusion:**
```diff
- "demonstrates that CogniMaps can eliminate fabricated retrieval dates"
- "will be essential for maintaining trust"

+ "In preliminary testing (n=15 questions, Gemini 2.0 Flash)..."
+ "This is early work with significant limitations (see §9)"
+ "If the approach proves robust under broader evaluation..."
+ "We welcome feedback from the research community"
```

---

### README (`README.md`)

#### **Header & Badges:**
```diff
- "Transform LLMs from 'confident but unverifiable' to 'confident AND auditable'"
- Status: Production Ready (green)

+ "Preliminary work on enforcing provenance requirements..."
+ Status: Experimental (yellow)
+ Added: Feedback Welcome (blue)
```

#### **Added Prominent Warning Box:**
```markdown
> ⚠️ Early Work / Seeking Feedback
>
> This is a preliminary prototype demonstrating structured provenance
> enforcement for LLM outputs in legal domains. Our evaluation
> consisted of 15 questions on one model (Gemini 2.0 Flash).
> Broader validation across models, domains, and sample sizes
> is needed before production use.
>
> Key Limitations:
> - Small sample size (n=15, no statistical testing)
> - Single model evaluation (Gemini only)
> - No ablation study (improvement may be due to prompting)
> - No ground truth verification (fee discrepancy unresolved)
> - Single domain (US patent law only)
```

#### **Results Section:**
```diff
- "Proven Results"
- "100% citation compliance" → "✅ +40%"
- "Key Finding: CogniMap adds structured provenance"

+ "Initial Observations"
+ "Status: Preliminary / Not statistically validated"
+ "Structured citations present" (no percentage claim)
+ "Initial Finding: ...However, n=15 is insufficient for statistical claims."
```

#### **Impact Section:**
```diff
- "Real-World Impact"
- "Result: Reduced filing errors, fewer deadline mistakes"

+ "Potential Impact"
+ "Hypothetical Scenario (Illustrative):"
+ "Note: Real-world impact depends on deployment context and broader evaluation"
```

#### **Removed:**
- "Built with ❤️" footer
- Excessive emojis from section headers (kept a few for readability)
- Definitive claims without hedging
- Marketing language ("Transform", "Proven", "Production-Ready")

#### **Changelog:**
```diff
- v1.0.0 (January 2026)
- 🎉 Initial public release

+ v0.1 (January 2026)
+ Initial public release (experimental)
+ Test suite with 15 questions (preliminary evaluation)
```

---

## What This Achieves

### **For Peer Reviewers:**
✅ Shows we understand limitations
✅ Invites critique rather than claiming perfection
✅ Acknowledges prior work (not claiming to have invented structured output)
✅ Provides concrete future work roadmap

### **For Practitioners:**
✅ Honest about what's validated vs. hypothetical
✅ Clear about single-model, small-n evaluation
✅ Transparent about ground truth uncertainty
✅ Realistic about operational burden

### **For Engineers:**
✅ No "production-ready" claims without evidence
✅ Acknowledges prompt engineering confound
✅ Recognizes need for ablation study
✅ Realistic about what n=15 can and cannot show

### **For Potential Users:**
✅ Clear this is experimental work
✅ Links to detailed limitations discussion
✅ Invitation to provide feedback
✅ No overselling capabilities

---

## What Remains Authentic

Despite the humility additions, the core contribution is real:

1. **Dual-registry architecture** is a genuine design pattern
2. **Fail-closed enforcement** is a valid safety principle
3. **Structured provenance blocks** are demonstrably produced
4. **Post-answer validation gates** are a real implementation
5. **The comparison** (n=15) genuinely shows structured vs. unstructured outputs

**We're not hiding the work; we're accurately scoping it.**

---

## Ready For

- ✅ HackerNews "Show HN" (technical audience will appreciate honesty)
- ✅ arXiv preprint (meets academic standards for limitations discussion)
- ✅ GitHub public release (demonstrates serious engineering)
- ✅ Practitioner feedback (invites collaboration vs. claiming expertise)
- ✅ Peer review (anticipates and addresses threats to validity)

---

## Not Ready For (Yet)

- ❌ Patent filing (need provisional first)
- ❌ "Production deployment" claims (need more evaluation)
- ❌ Commercial licensing (experimental status)
- ❌ Academic conference submission (need n>100, multi-model)

---

## Next Steps

**To strengthen credibility further:**

1. **File provisional patent** (establishes priority date)
2. **Expand evaluation:**
   - n=100+ questions
   - 3+ models (GPT-4, Claude, Llama)
   - 2+ domains (medical, financial)
3. **Ablation study:**
   - CogniMap with gates disabled
   - CogniMap with schemas only
   - Matched prompt length baseline
4. **Ground truth verification:**
   - Manual verification of 50 claims by domain expert
   - Document discrepancies
5. **User study:**
   - 10-20 patent attorneys/agents
   - Qualitative feedback on utility
6. **Red-team testing:**
   - Adversarial prompts to bypass gates
   - Edge case discovery

---

**Summary:** This is now positioned as serious preliminary work with appropriate humility, inviting collaboration rather than claiming perfection. The technical contribution is real; the scoping is honest.
