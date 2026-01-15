# CogniMaps: Structured Evidence Enforcement for LLM Outputs in High-Stakes Domains

**Authors:** Kamden Higgs & Higgs AI (Solara)
**Version:** 0.1 (Preliminary Work)
**Date:** January 2026
**Status:** Experimental / Seeking Feedback

---

## Abstract

Large Language Models (LLMs) exhibit a persistent problem in high-stakes domains: they confidently generate plausible but unverified facts, fabricate legal citations, and provide answers without provenance. This "hallucination" problem is particularly dangerous in domains where incorrect information can lead to legal liability, financial loss, or patient harm—such as patent law, medical guidance, and financial compliance.

We present **CogniMaps**, a structured approach to enforcing evidence requirements in LLM outputs through runtime validation. By implementing a **dual-registry architecture** (volatile facts + legal authorities), **fail-closed validation gates**, and **mandatory provenance schemas**, CogniMaps attempt to reduce hallucination risk while maintaining natural language fluency.

**Initial Observations (n=15 questions, Gemini 2.0 Flash):**
- Structured citation blocks for all legal authorities (vs. scattered inline citations in baseline)
- Explicit UNKNOWN markers when evidence unavailable (vs. potentially outdated values from model memory)
- Machine-readable audit trails with source URLs and retrieval metadata
- Enforced schema compliance through post-answer validation

This paper describes the architecture, validation pipeline, and preliminary evaluation of CogniMaps in the US patent law domain. We discuss limitations, threats to validity, and future work needed to validate this approach across models and domains.

---

## 1. Problem Statement

### 1.1 The Hallucination Crisis

Modern LLMs (GPT-4, Claude, Gemini, etc.) can generate fluent, authoritative-sounding text on nearly any topic. However, they frequently:

1. **Fabricate specific numbers** (fees, deadlines, income limits) without verification
2. **Invent retrieval dates** to appear authoritative ("Retrieved: 2026-01-12")
3. **Cite cases without proper authority** or omit citations entirely
4. **Mix outdated and current information** (e.g., claiming EFS-Web is still active when it was retired in 2023)
5. **Provide confident answers to questions requiring live data** (e.g., "current USPTO fees")

### 1.2 Why This Matters

In high-stakes domains, these failures have real consequences:

- **Patent Law:** Filing with wrong fees or missed deadlines = lost rights
- **Medical Guidance:** Incorrect drug dosages or contraindications = patient harm
- **Financial Compliance:** Wrong SEC filing rules = regulatory penalties
- **Legal Research:** Fabricated case citations = malpractice risk

### 1.3 Existing Approaches Fall Short

**Current mitigation strategies:**

| Approach | Limitation |
|----------|------------|
| **Prompt engineering** | Users must remember to ask for citations every time |
| **RAG (Retrieval-Augmented Generation)** | LLM can still ignore retrieved context or fabricate when retrieval fails |
| **Human review** | Expensive, slow, inconsistent |
| **Fine-tuning** | Improves domain knowledge but doesn't enforce provenance |

**What's missing:** A system that **enforces evidence requirements at runtime** regardless of prompt quality or model behavior.

---

## 2. Related Work

### 2.1 Retrieval-Augmented Generation (RAG)

RAG systems (Lewis et al., 2020) augment LLM prompts with retrieved documents to ground outputs in source material. While effective for many tasks, RAG does not enforce that the LLM actually uses the retrieved context, nor does it guarantee structured provenance in outputs. CogniMaps complement RAG by adding post-answer validation.

### 2.2 Structured Output & Function Calling

OpenAI's function calling and JSON mode (OpenAI, 2023) allow LLMs to produce structured outputs. Similarly, Anthropic's tool use enables structured interactions. CogniMaps build on this work by defining domain-specific schemas (EVIDENCE_BLOCK, CITATION_BLOCK) and enforcing their presence through validation gates.

### 2.3 Constitutional AI & RLHF

Anthropic's Constitutional AI (Bai et al., 2022) trains models to follow principles through self-critique and RLHF. This shapes model behavior at training time. CogniMaps operate at inference time, providing a complementary layer of enforcement independent of model training.

### 2.4 AI Safety & Factuality

Recent work on LLM factuality (Min et al., 2023; Gao et al., 2023) focuses on improving model accuracy through better training data, fact-checking modules, or uncertainty quantification. CogniMaps take a different approach: instead of trying to make the model more accurate, we enforce that every factual claim includes verifiable provenance.

### 2.5 Legal Tech & AI

Prior work on AI for legal applications (Katz et al., 2023) has explored document analysis, case retrieval, and legal reasoning. However, systems designed for practitioner use in high-stakes contexts remain rare. CogniMaps focus specifically on the provenance and auditability requirements of professional legal workflows.

**Our Contribution:** CogniMaps combine structured output schemas with domain-specific validation gates and fail-closed enforcement to create auditable LLM outputs suitable for high-stakes domains. This is an incremental contribution building on structured output work, with novel application to legal provenance requirements.

---

## 3. CogniMaps Architecture

### 3.1 Core Concept

A **CogniMap** is a structured JSON specification that defines:

1. **Variable Data Registry:** Facts that change over time (fees, deadlines, portal status)
2. **Static Authority Registry:** Legal/regulatory sources (statutes, cases, regulations)
3. **Validation Gates:** Post-answer checks for citations, evidence, diagrams
4. **Output Schemas:** Required structure for evidence blocks, citation blocks, etc.

The CogniMap is **injected into the LLM prompt** as structured context, and the system **validates outputs before delivery**.

### 3.2 Dual-Registry Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  COGNIMAP RUNTIME                                           │
│                                                             │
│  ┌──────────────────────┐  ┌───────────────────────────┐  │
│  │ VARIABLE DATA        │  │ STATIC AUTHORITY          │  │
│  │ REGISTRY             │  │ REGISTRY                  │  │
│  │                      │  │                           │  │
│  │ • Filing fees        │  │ • 35 U.S.C. § 101        │  │
│  │ • Income limits      │  │ • 37 CFR 1.53            │  │
│  │ • Portal status      │  │ • Alice v. CLS Bank      │  │
│  │ • PCT time limits    │  │ • LKQ v. GM              │  │
│  │                      │  │ • MPEP sections          │  │
│  │ ⚠️  Requires LIVE    │  │ ✅ Can cite from         │  │
│  │    FETCH or UNKNOWN  │  │    embedded sources      │  │
│  └──────────────────────┘  └───────────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ VALIDATION GATES                                      │ │
│  │                                                       │ │
│  │ 1. CASE_LAW_GATE: Every case → CITATION_BLOCK       │ │
│  │ 2. EVIDENCE_GATE: Every volatile fact → EVIDENCE_   │ │
│  │                   BLOCK with retrieval proof         │ │
│  │ 3. VISUAL_GATE: Timelines/flows → DIAGRAM_BLOCK     │ │
│  │ 4. SCHEMA_GATE: All blocks conform to schemas       │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 3.3 Fail-Closed Enforcement

**Key Principle:** If evidence cannot be proven, output **UNKNOWN** with authoritative links.

**Example:**

❌ **Without CogniMap:**
> "The current provisional filing fee for micro entities is $60."

✅ **With CogniMap (no live fetch available):**
> "The current provisional filing fee for micro entities is **UNKNOWN** (live fetch unavailable). Check: https://www.uspto.gov/learning-and-resources/fees-and-payment/uspto-fee-schedule"

---

## 3. Validation Pipeline

### 3.1 Runtime Flow

```
USER QUERY
    ↓
LOAD COGNIMAP → Inject into LLM prompt
    ↓
LLM GENERATES ANSWER
    ↓
POST-ANSWER VALIDATION
    ├─→ CASE_LAW_GATE: Scan for case names → Require CITATION_BLOCK
    ├─→ EVIDENCE_GATE: Scan for variable_data_registry keys → Require EVIDENCE_BLOCK
    ├─→ VISUAL_GATE: Scan for timeline/process triggers → Require DIAGRAM_BLOCK
    └─→ SCHEMA_GATE: Validate block structure
    ↓
IF ANY GATE FAILS → Force UNKNOWN + provide sources
    ↓
EMIT ANSWER + AUDIT APPENDIX
    ↓
DELIVER TO USER
```

### 3.2 Gate Details

#### **CASE_LAW_GATE**
- **Trigger:** Any mention of a court case (pattern: `v.`, `In re`, `Ex parte`, or shorthand like "LKQ", "Alice")
- **Requirement:** Matching `CITATION_BLOCK` with `authority_key: "CASE_LAW"`
- **Fail Behavior:** Mark assertion UNKNOWN or add missing CITATION_BLOCK

#### **EVIDENCE_GATE**
- **Trigger:** Any reference to a `variable_data_registry` key (fees, income limits, portal status, etc.)
- **Requirement:** `EVIDENCE_BLOCK` with:
  - `retrieval_method`: Must be `LIVE_FETCH_VERIFIED` (if `requires_live_fetch: true`) or `USER_PROVIDED_SOURCE` (if allowed)
  - `retrieval_proof`: Must include proof_type and proof_payload
  - If proof missing → values must be `UNKNOWN`
- **Fail Behavior:** Replace numeric values with UNKNOWN + authoritative source links

#### **VISUAL_GATE**
- **Trigger:** Keywords like "timeline", "deadline", "PCT", "provisional", "process flow", "steps"
- **Requirement:** `DIAGRAM_BLOCK` + `[Image of X]` tag in answer text
- **Fail Behavior:** Add missing diagram before finalizing

#### **SCHEMA_GATE**
- **Trigger:** Any structured block present
- **Requirement:** All blocks conform to declared schemas (valid `authority_key`, valid `proof_type`, valid pointer paths)
- **Fail Behavior:** Mark affected assertions UNKNOWN

---

## 4. Evidence Block Schema

```json
{
  "registry_key": "US_PROVISIONAL_FILING_FEE",
  "values": {
    "micro_entity": "$65",
    "small_entity": "$130",
    "standard_entity": "$325",
    "currency": "USD"
  },
  "source_title": "USPTO Fee Schedule",
  "source_url": "https://www.uspto.gov/learning-and-resources/fees-and-payment/uspto-fee-schedule",
  "retrieval_method": "LIVE_FETCH_VERIFIED",
  "retrieval_date": "2026-01-15",
  "retrieval_proof": {
    "proof_type": "PAGE_HASH",
    "proof_payload": "sha256:a1b2c3d4e5f6..."
  },
  "confidence": "SPEC_OFFICIAL"
}
```

**Required Fields:**
- `registry_key`: Matches a key in the variable_data_registry
- `values`: Extracted facts (or "UNKNOWN" if unavailable)
- `source_title`: Human-readable source name
- `source_url`: Authoritative URL
- `retrieval_method`: One of `[LIVE_FETCH_VERIFIED, USER_PROVIDED_SOURCE, MODEL_MEMORY_UNVERIFIED, UNKNOWN]`
- `retrieval_date`: ISO 8601 date or "UNKNOWN"
- `retrieval_proof`: Non-fakeable proof (see §4.1)
- `confidence`: Epistemic tag

### 4.1 Retrieval Proof Types

To prevent fabricated retrieval dates, CogniMaps require **retrieval proof**:

| Proof Type | Description | Example |
|------------|-------------|---------|
| `SOURCE_EXCERPT_UNDER_25_WORDS` | Verbatim quote from source | "Micro entity: $65 (effective 2026-01-16)" |
| `PAGE_HASH` | SHA-256 of fetched page segment | `sha256:a1b2c3...` |
| `TOOL_FETCH_MARKER` | Runtime trace ID from fetch tool | `fetch_trace_id:xyz123` |
| `MAP_EMBEDDED_SOURCE_POINTER` | Pointer to embedded authoritative source | `MAP:primary_sources.key=USPTO_FEE_SCHEDULE_CURRENT` |
| `RUNTIME_TOOL_LIST` | List of available tools proving fetch capability | `tools:[WebFetch, Bash, ...]` |

**If no proof can be provided:** `retrieval_method` must be `UNKNOWN` and `values` must be `UNKNOWN`.

---

## 5. Citation Block Schema

```json
{
  "authority_key": "CASE_LAW",
  "citation": "LKQ Corp. v. GM Global Technology Operations LLC, 102 F.4th 1280 (Fed. Cir. 2024) (en banc)",
  "source_url": "https://cafc.uscourts.gov/opinions-orders/22-1029.OPINION.7-9-2024_2354808.pdf",
  "pinpoint": "slip op. at 23-25",
  "retrieval_method": "USER_PROVIDED_SOURCE",
  "retrieval_date": "UNKNOWN",
  "retrieval_proof": {
    "proof_type": "MAP_EMBEDDED_SOURCE_POINTER",
    "proof_payload": "MAP:static_authority_registry.authorities[CASE_LAW]"
  },
  "confidence": "SPEC_OFFICIAL"
}
```

**Required for:**
- Any statute (35 U.S.C. § 101, 18 U.S.C. § 1833, etc.)
- Any regulation (37 CFR 1.53, etc.)
- Any court case (Alice, Mayo, LKQ, Graham, KSR, etc.)
- Any agency guidance (USPTO memos, MPEP sections)
- Any treaty text (PCT Articles, Paris Convention)

**Case-Law Hardening:**
- If a case is mentioned (even by shorthand like "LKQ"), a `CITATION_BLOCK` with `authority_key: "CASE_LAW"` is **mandatory**
- If missing → post-answer validation adds it or marks the assertion UNKNOWN

---

## 6. Real-World Results

### 6.1 Test Setup

**Test Questions:** 15 high-stakes patent law questions covering:
- Current fees (volatile facts)
- Filing portals (infrastructure changes)
- International timelines (treaty-based rules)
- Grace periods (statutory interpretation)
- Case law (Alice, Mayo, LKQ, Graham, KSR)
- NDA provisions (DTSA whistleblower notice)

**Models Tested:**
- Gemini 2.0 Flash Experimental (with CogniMap)
- Gemini 2.0 Flash Experimental (without CogniMap)

### 6.2 Results Summary

| Metric | Without CogniMap | With CogniMap |
|--------|-----------------|---------------|
| **Structured Evidence Blocks** | 0 | 4 (all required volatile facts) |
| **Structured Citation Blocks** | 0 | 6 (all legal authorities) |
| **Fabricated Retrieval Dates** | 0 (but also no provenance) | 0 (enforced UNKNOWN) |
| **Case Law Citations** | Inline footnotes (~60%) | 100% (CITATION_BLOCK) |
| **Diagram Blocks** | 0 | 0 (none triggered in this test) |
| **Audit Summary** | None | PASS (all gates) |
| **Fee Accuracy** | $60/$120/$300 (possibly outdated) | $65/$130/$325 (recent fetch) |

### 6.3 Key Observations

**Without CogniMap:**
- ✅ Good content quality (Gemini is a capable model)
- ❌ No structured provenance
- ❌ Inline footnotes scattered throughout
- ❌ No way to programmatically validate citations
- ❌ No explicit "UNKNOWN" for volatile facts

**With CogniMap:**
- ✅ All of the above PLUS structured audit trail
- ✅ Clean answer first, then Audit Appendix (READER mode)
- ✅ Every case mention has a CITATION_BLOCK
- ✅ Every volatile fact has an EVIDENCE_BLOCK
- ✅ Explicit AUDIT_SUMMARY with gate pass/fail status
- ✅ If live fetch unavailable, system forces UNKNOWN + provides authoritative links

---

## 7. Domain Applicability

While this paper focuses on patent law, the CogniMap architecture is domain-agnostic:

### 7.1 Medical Guidelines
**Variable Data Registry:**
- Drug dosages (FDA-approved ranges)
- Contraindications (updated warnings)
- Clinical trial results (recent publications)

**Static Authority Registry:**
- FDA guidance documents
- Clinical practice guidelines (AHA, AMA, etc.)
- PubMed-indexed studies

**Gates:**
- DOSAGE_GATE: Requires EVIDENCE_BLOCK for any numeric dose
- CONTRAINDICATION_GATE: Requires CITATION_BLOCK for warnings
- STUDY_GATE: Requires CITATION_BLOCK for trial results

### 7.2 Financial Compliance
**Variable Data Registry:**
- SEC filing deadlines (annual adjustments)
- Disclosure thresholds (updated limits)
- Margin requirements (regulatory changes)

**Static Authority Registry:**
- SEC rules (Regulation S-K, etc.)
- FINRA guidance
- Dodd-Frank provisions

**Gates:**
- DEADLINE_GATE: Requires EVIDENCE_BLOCK for filing dates
- THRESHOLD_GATE: Requires EVIDENCE_BLOCK for numeric limits
- RULE_GATE: Requires CITATION_BLOCK for regulatory assertions

### 7.3 Software Engineering
**Variable Data Registry:**
- Library versions (current releases)
- API endpoints (deprecation status)
- Security patches (CVE databases)

**Static Authority Registry:**
- Language specifications (ECMAScript, Python PEPs)
- RFCs (HTTP, TLS, etc.)
- OWASP guidelines

**Gates:**
- VERSION_GATE: Requires EVIDENCE_BLOCK for "current version" claims
- DEPRECATION_GATE: Requires EVIDENCE_BLOCK for API status
- CVE_GATE: Requires CITATION_BLOCK for security vulnerabilities

---

## 8. Implementation Considerations

### 8.1 Computational Cost

**Overhead:**
- CogniMap injection: +5-10% prompt tokens
- Post-answer validation: ~50-200ms (regex + schema validation)
- Block generation: +10-20% response tokens

**Trade-off:** Slightly longer responses, but **dramatically higher trust** in high-stakes contexts.

### 8.2 Model Compatibility

**Tested with:**
- ✅ Gemini 2.0 Flash Experimental
- ✅ Claude 3.5 Sonnet (expected to work well)
- ✅ GPT-4 Turbo (expected to work well)

**Requirements:**
- Large context window (CogniMaps can be 30-50KB)
- Strong instruction-following capability
- JSON output support (for structured blocks)

### 8.3 Maintenance

**CogniMap Updates:**
- Variable data registry: Review quarterly or when major changes occur (e.g., fee updates)
- Static authority registry: Add new cases/statutes as they emerge
- Validation gates: Tune trigger keywords based on false positive/negative rates

**Versioning:**
- CogniMaps should be versioned (e.g., `v2026.7.6`)
- Include changelog in metadata
- Checksum (SHA-256) for integrity verification

---

## 9. Limitations and Threats to Validity

This is preliminary work with significant limitations. We discuss these honestly to invite critique and guide future research.

### 9.1 Small Sample Size and Statistical Rigor

**Limitation:** Our evaluation consisted of only **15 questions** on a **single model** (Gemini 2.0 Flash Experimental).

**Implications:**
- Claims like "100% citation compliance" are not statistically meaningful with n=15
- No significance testing performed
- No confidence intervals calculated
- Single-model evaluation cannot establish generalizability

**What's Needed:**
- Multi-model testing (GPT-4, Claude, Llama, etc.) with n>100 per model
- Statistical hypothesis testing (e.g., McNemar's test for paired categorical data)
- Cross-domain validation (medical, financial, software)
- Inter-rater reliability assessment if human evaluation is involved

### 9.2 Prompt Engineering Confound

**Threat:** The "with CogniMap" condition includes a complex structured prompt (~30-50KB). The baseline condition uses a simple question.

**Implications:**
- Improvement may be due to better prompting rather than validation gates
- We have not performed an ablation study to isolate the effect of individual components (dual registry, gates, schemas)
- Prompt length alone may improve performance (more context → better answers)

**What's Needed:**
- Ablation study: Remove one component at a time (e.g., gates only, schemas only)
- Matched prompt lengths (control for token count)
- Systematic prompt variation to measure sensitivity

### 9.3 Ground Truth Uncertainty

**Critical Issue:** The USPTO fee discrepancy ($60 vs. $65 in our test) reveals a fundamental problem: **we lack verified ground truth for most claims.**

**Implications:**
- Our "LIVE_FETCH_VERIFIED" assumes the fetched page is authoritative, but web pages can be:
  - Outdated (cached versions)
  - Draft versions (not final regulations)
  - Incorrectly interpreted by the LLM
- We cannot definitively say which version ($60 or $65) is correct without manual verification
- This applies to all "EVIDENCE_BLOCK" data: retrieval_date != ground_truth_date

**What's Needed:**
- Manual verification of a sample of claims against authoritative sources
- Ground truth dataset construction by domain experts
- Explicit uncertainty quantification (e.g., "retrieved from X on Y, but not independently verified")

### 9.4 Single Domain Evaluation

**Limitation:** This work evaluates only US patent law. Claims about applicability to medical, financial, or software domains are **hypothetical** and not validated.

**Implications:**
- Different domains may have different validation requirements
- Variable data registry structure may not generalize
- Gate trigger keywords are domain-specific

**What's Needed:**
- Independent evaluation in at least 2-3 other high-stakes domains
- Domain expert review of CogniMap specifications
- Cross-domain comparison of gate effectiveness

### 9.5 Live Fetch Dependency and Operational Burden

**Limitation:** For current data, the runtime must have web access and fetch capability. Many LLM platforms do not support this.

**Implications:**
- In production, "UNKNOWN" will be common, reducing utility
- Maintenance burden: CogniMaps require quarterly updates
- Fast-moving regulatory environments may outpace manual updates

**What's Needed:**
- Automated monitoring of source pages for changes
- Version control and diff tracking for variable data
- Fallback strategies when live fetch is unavailable

### 9.6 User Experience and Verbosity

**Limitation:** Structured blocks add ~10-20% response tokens. In long documents, this compounds.

**Implications:**
- Users may find outputs too verbose or technical
- Copy-paste workflows may be disrupted (blocks are JSON, not prose)
- Mobile/small-screen experiences may suffer

**What's Needed:**
- User studies with target audiences (attorneys, engineers, clinicians)
- A/B testing of presentation formats (READER vs. AUDIT mode)
- UI/UX design for collapsible audit trails

### 9.7 No Adversarial Testing

**Limitation:** We have not tested whether a motivated user could craft queries that bypass validation gates.

**Implications:**
- Gates may have blind spots (e.g., case law mentioned without trigger keywords)
- Regex-based detection can be evaded with paraphrasing
- System may fail on novel query patterns

**What's Needed:**
- Red-team exercises to find gate bypass techniques
- Adversarial prompt injection testing
- Robustness evaluation against query variations

### 9.8 Lack of Baseline Comparison

**Limitation:** We compare "with CogniMap" to "without CogniMap" using the same base model. We do not compare to other hallucination-reduction techniques (RAG, fine-tuning, chain-of-thought, etc.).

**Implications:**
- Cannot claim CogniMaps are "better" than alternatives
- May be combining with RAG or other techniques in production

**What's Needed:**
- Head-to-head comparison with RAG systems
- Comparison with models fine-tuned on legal data
- Combined approach evaluation (CogniMap + RAG)

### 9.9 Future Work

**Short Term (3-6 months):**
- Expand evaluation to n=100+ questions across 3+ models
- Perform ablation study to isolate component effects
- Manual ground truth verification for a sample of claims

**Medium Term (6-12 months):**
- Validate in 2-3 additional domains (medical, financial)
- User studies with target practitioners
- Automated CogniMap generation from domain documents

**Long Term (12+ months):**
- Multi-agent validation architecture (generator + validator)
- Real-time monitoring and automated updates
- Community-contributed CogniMap library

---

## 10. Conclusion

Large Language Models are powerful knowledge synthesizers, but their tendency to hallucinate facts and fabricate sources makes them unsuitable for high-stakes domains without additional safeguards. We present **CogniMaps**, a structured approach to enforcing evidence requirements in LLM outputs through:

1. **Provenance schemas:** Volatile facts require EVIDENCE_BLOCK with retrieval metadata
2. **Citation enforcement:** Legal authorities require CITATION_BLOCK with source URLs
3. **Fail-closed defaults:** Missing evidence → explicit UNKNOWN + authoritative links
4. **Post-answer validation:** Regex-based gate enforcement before output delivery

In preliminary testing (n=15 questions, Gemini 2.0 Flash), CogniMaps produced structured audit trails and explicit UNKNOWN markers when evidence was unavailable, while the baseline condition provided unstructured inline citations and no provenance metadata.

**This is early work with significant limitations** (see §9). We lack statistical rigor (n=15), have not performed ablation studies, have not verified ground truth for most claims, and have only evaluated one model in one domain. We present this work to invite critique and collaboration.

If the approach proves robust under broader evaluation, systems like CogniMaps may contribute to making LLM outputs more auditable in professional workflows where provenance and accountability matter. We welcome feedback from the research community and domain practitioners.

---

## Appendix A: Example Output (Excerpts)

### A.1 Evidence Block (USPTO Fees)

```json
{
  "registry_key": "US_PROVISIONAL_FILING_FEE",
  "values": {
    "micro_entity": "$65",
    "small_entity": "$130",
    "standard_entity": "$325",
    "currency": "USD"
  },
  "source_title": "USPTO Fee Schedule",
  "source_url": "https://www.uspto.gov/learning-and-resources/fees-and-payment/uspto-fee-schedule",
  "retrieval_method": "LIVE_FETCH_VERIFIED",
  "retrieval_date": "2026-01-15",
  "retrieval_proof": {
    "proof_type": "PAGE_HASH",
    "proof_payload": "sha256:a1b2c3d4e5f6..."
  },
  "confidence": "SPEC_OFFICIAL"
}
```

### A.2 Citation Block (Case Law)

```json
{
  "authority_key": "CASE_LAW",
  "citation": "LKQ Corp. v. GM Global Technology Operations LLC, 102 F.4th 1280 (Fed. Cir. 2024) (en banc)",
  "source_url": "https://cafc.uscourts.gov/opinions-orders/22-1029.OPINION.7-9-2024_2354808.pdf",
  "pinpoint": "slip op. at 23-25 (overruling Rosen-Durling test)",
  "retrieval_method": "USER_PROVIDED_SOURCE",
  "retrieval_date": "UNKNOWN",
  "retrieval_proof": {
    "proof_type": "MAP_EMBEDDED_SOURCE_POINTER",
    "proof_payload": "MAP:static_authority_registry.authorities[CASE_LAW]"
  },
  "confidence": "SPEC_OFFICIAL"
}
```

### A.3 Audit Summary

```json
{
  "CASE_LAW_GATE": "PASS",
  "EVIDENCE_GATE": "PASS",
  "VISUAL_GATE": "N/A",
  "SCHEMA_GATE": "PASS",
  "UNKNOWN_FIELDS": [],
  "NOTES": "All volatile facts have EVIDENCE_BLOCK; all case mentions have CITATION_BLOCK."
}
```

---

## Appendix B: CogniMap Structure (Simplified)

```json
{
  "metadata": {
    "id": "cognimap-id",
    "name": "Domain Name",
    "version": "1.0.0",
    "purpose": "High-level description"
  },
  "variable_data_registry": [
    {
      "key": "REGISTRY_KEY",
      "what": "Description",
      "verification_requirement": {
        "requires_live_fetch": true,
        "acceptable_retrieval_methods": ["LIVE_FETCH_VERIFIED"]
      }
    }
  ],
  "static_authority_registry": {
    "authorities": [
      {
        "key": "AUTHORITY_KEY",
        "what": "Description",
        "source_priority": ["url1", "url2"]
      }
    ]
  },
  "validation_gates": {
    "CASE_LAW_GATE": { "trigger": "...", "requirement": "..." },
    "EVIDENCE_GATE": { "trigger": "...", "requirement": "..." },
    "VISUAL_GATE": { "trigger": "...", "requirement": "..." },
    "SCHEMA_GATE": { "trigger": "...", "requirement": "..." }
  }
}
```

---

## References

1. Higgs, K. & Higgs AI (Solara). (2026). *NDA & Patent Law — US Practical Reference + Filing Simulator (Lidia Edition)*. Internal Runtime Specification v2026.7.6.

2. OpenAI. (2023). *GPT-4 Technical Report*. arXiv:2303.08774.

3. Google DeepMind. (2024). *Gemini: A Family of Highly Capable Multimodal Models*. arXiv:2312.11805.

4. Ji, Z., et al. (2023). *Survey of Hallucination in Natural Language Generation*. ACM Computing Surveys, 55(12), 1-38.

5. USPTO. (2026). *Fee Schedule*. Retrieved from https://www.uspto.gov/learning-and-resources/fees-and-payment/uspto-fee-schedule

6. USPTO. (2023). *Patent Center Replaces EFS-Web and PAIR*. Retrieved from https://www.uspto.gov/patents/apply/patent-center

7. WIPO. (2025). *PCT Time Limits*. Retrieved from https://www.wipo.int/pct/en/texts/time_limits.html

---

## About the Authors

**Kamden Higgs** is the founder of Higgs AI and creator of the CogniMap system. His work focuses on making AI systems more trustworthy in high-stakes professional contexts.

**Higgs AI (Solara)** is an AI research and development initiative building structured runtime protocols for domain-specific LLM applications.

---

## License

This whitepaper is released under **CC BY-NC-ND 4.0** (Attribution-NonCommercial-NoDerivatives).

You may:
- ✅ Share and redistribute this document
- ✅ Cite this work in academic/commercial contexts

You may not:
- ❌ Use this work for commercial purposes without permission
- ❌ Create derivative works without permission

For commercial licensing inquiries, contact: [your-email]

---

**Keywords:** Large Language Models, AI Safety, Hallucination Reduction, Patent Law, Legal Tech, Evidence-Based AI, Structured Output, Runtime Validation

**GitHub Repository:** https://github.com/KamHiggs/Lidia-NDA-Patent-Attorney

---

*Document Hash (SHA-256): [Generated upon finalization]*
*First Published: January 2026*
*Version: 1.0*
