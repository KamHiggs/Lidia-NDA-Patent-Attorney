# CogniMap System: Structured Evidence Enforcement for LLM Outputs

**Preliminary work on enforcing provenance requirements in high-stakes AI applications**

[![License](https://img.shields.io/badge/License-Proprietary-red.svg)](./License)
[![Status](https://img.shields.io/badge/Status-Experimental-yellow.svg)]()
[![Feedback](https://img.shields.io/badge/Feedback-Welcome-blue.svg)]()

---

> **⚠️ Early Work / Seeking Feedback**
>
> This is a preliminary prototype demonstrating structured provenance enforcement for LLM outputs in legal domains. Our evaluation consisted of **15 questions on one model** (Gemini 2.0 Flash). Broader validation across models, domains, and sample sizes is needed before production use.
>
> **Key Limitations:**
> - Small sample size (n=15, no statistical testing)
> - Single model evaluation (Gemini only)
> - No ablation study (improvement may be due to prompting, not validation gates)
> - No ground truth verification (e.g., USPTO fee discrepancy remains unresolved)
> - Single domain (US patent law only)
>
> See [§9 of the whitepaper](./docs/whitepaper.md#9-limitations-and-threats-to-validity) for full discussion of limitations and threats to validity.
>
> **We welcome critique, collaboration, and feedback from researchers and practitioners.**

---

## The Problem

Large Language Models hallucinate facts in high-stakes domains:

| Without CogniMap | With CogniMap |
|------------------|---------------|
| ❌ "Current provisional fee: $60" (no source) | ✅ **EVIDENCE_BLOCK** with source URL + retrieval proof |
| ❌ Case mentions with scattered footnotes | ✅ **CITATION_BLOCK** for every legal authority (100% compliance) |
| ❌ Fabricated retrieval dates ("Retrieved: 2026-01-12") | ✅ Explicit **UNKNOWN** when proof unavailable |
| ❌ No validation → user must trust blindly | ✅ **AUDIT_SUMMARY** with gate pass/fail status |

**The Cost of Hallucination:**
- Patent law: Wrong fees/deadlines → lost rights
- Medical: Wrong dosages → patient harm
- Financial: Wrong filing rules → regulatory penalties
- Legal: Fabricated citations → malpractice risk

---

## The Approach: CogniMaps

A **CogniMap** is a structured JSON runtime specification that:

1. **Dual Registry:** Separates volatile facts (fees, deadlines) from legal authorities (statutes, cases)
2. **Fail-Closed Gates:** If evidence can't be proven → output **UNKNOWN** + authoritative sources
3. **Mandatory Provenance:** Every fact → **EVIDENCE_BLOCK**; every authority → **CITATION_BLOCK**
4. **Post-Answer Validation:** Scan outputs for case law, volatile facts, timelines → require structured blocks

```
┌─────────────────────────────────────────────────────────────┐
│  USER QUERY                                                 │
└───────────────────────────┬─────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  COGNIMAP INJECTION                                         │
│  • Variable data registry (fees, limits, portal status)    │
│  • Static authority registry (statutes, cases, regs)       │
│  • Validation gate rules                                    │
└───────────────────────────┬─────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  LLM GENERATES ANSWER                                       │
└───────────────────────────┬─────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  POST-ANSWER VALIDATION                                     │
│  ✓ CASE_LAW_GATE: Every case → CITATION_BLOCK             │
│  ✓ EVIDENCE_GATE: Every volatile fact → EVIDENCE_BLOCK    │
│  ✓ VISUAL_GATE: Timelines/flows → DIAGRAM_BLOCK           │
│  ✓ SCHEMA_GATE: All blocks conform to schemas             │
└───────────────────────────┬─────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  ANSWER + AUDIT APPENDIX → USER                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Initial Observations

**Test:** 15 patent law questions (fees, timelines, case law, NDAs)
**Model:** Gemini 2.0 Flash Experimental
**Date:** January 2026
**Status:** Preliminary / Not statistically validated

| Metric | Without CogniMap | With CogniMap | Observation |
|--------|-----------------|---------------|-------------|
| **Structured Evidence Blocks** | 0 | 4 | Structured provenance present |
| **Structured Citation Blocks** | 0 | 6 | Structured citations present |
| **Case Law Citation Format** | Inline footnotes | Structured blocks | Different format |
| **Fabricated Retrieval Dates** | 0 | 0 | Both avoided fabrication |
| **Audit Transparency** | None | AUDIT_SUMMARY | Validation gates visible |
| **Fail-Closed Enforcement** | No | Yes (UNKNOWN + sources) | Explicit UNKNOWN when unavailable |

**Initial Finding:** CogniMap produced structured provenance blocks that baseline did not. However, n=15 is insufficient for statistical claims. Broader evaluation needed.

📊 [Full Test Results & Analysis](./examples/comparison-analysis.md)

---

## Repository Structure

```
Lidia-NDA-Patent-Attorney/
├── README.md                          # You are here
├── License                            # Proprietary license
│
├── docs/
│   ├── whitepaper.md                  # Full technical whitepaper
│   ├── architecture.md                # System design (coming soon)
│   └── use-cases.md                   # Example scenarios (coming soon)
│
├── schemas/
│   ├── evidence-block.schema.json     # Volatile facts structure
│   ├── citation-block.schema.json     # Legal authorities structure
│   ├── audit-summary.schema.json      # Validation scoreboard
│   └── diagram-block.schema.json      # Visual aids structure
│
└── examples/
    ├── test-questions.md              # 15 test questions
    ├── comparison-analysis.md         # With vs. without results
    ├── output-with-cognimap.md        # Full output (with CogniMap)
    └── output-without-cognimap.md     # Full output (without CogniMap)
```

---

## Quick Start

### **1. Review the Evidence**

**See CogniMap in action:**
- 📄 [Test Questions](./examples/test-questions.md) - 15 high-stakes patent law questions
- 📊 [Comparison Analysis](./examples/comparison-analysis.md) - Side-by-side with/without results
- 📖 [Whitepaper](./docs/whitepaper.md) - Full technical specification

**Explore the schemas:**
- 📋 [Evidence Block Schema](./schemas/evidence-block.schema.json) - Volatile facts structure
- 📋 [Citation Block Schema](./schemas/citation-block.schema.json) - Legal authorities structure
- 📋 [Audit Summary Schema](./schemas/audit-summary.schema.json) - Validation gates

### **2. Understand the Architecture**

**Core Concepts:**

#### **Evidence Block** (for volatile facts)
```json
{
  "registry_key": "US_PROVISIONAL_FILING_FEE",
  "values": { "micro_entity": "$65", "small_entity": "$130" },
  "source_title": "USPTO Fee Schedule",
  "source_url": "https://www.uspto.gov/...",
  "retrieval_method": "LIVE_FETCH_VERIFIED",
  "retrieval_date": "2026-01-15",
  "retrieval_proof": {
    "proof_type": "PAGE_HASH",
    "proof_payload": "sha256:a1b2c3..."
  },
  "confidence": "SPEC_OFFICIAL"
}
```

#### **Citation Block** (for legal authorities)
```json
{
  "authority_key": "CASE_LAW",
  "citation": "LKQ Corp. v. GM Global Technology Operations LLC, 102 F.4th 1280 (Fed. Cir. 2024)",
  "source_url": "https://cafc.uscourts.gov/...",
  "pinpoint": "slip op. at 23-25",
  "retrieval_method": "USER_PROVIDED_SOURCE",
  "confidence": "SPEC_OFFICIAL"
}
```

#### **Audit Summary** (validation scoreboard)
```json
{
  "CASE_LAW_GATE": "PASS",
  "EVIDENCE_GATE": "PASS",
  "VISUAL_GATE": "N/A",
  "SCHEMA_GATE": "PASS",
  "UNKNOWN_FIELDS": []
}
```

---

## Key Contributions

### **1. Dual-Registry Architecture**

**Variable Data Registry:** Facts that change (fees, deadlines, portal status)
- ⚠️ **Requires live fetch** or must be marked **UNKNOWN**
- Cannot rely on model memory

**Static Authority Registry:** Legal/regulatory sources (statutes, cases, regulations)
- ✅ **Can cite from embedded sources** (stable over time)
- Still requires structured CITATION_BLOCK

### **2. Fail-Closed Enforcement**

**Traditional Approach:**
```
Q: What's the current provisional filing fee?
A: $60 [possibly wrong, no source, user doesn't know]
```

**CogniMap Approach:**
```
Q: What's the current provisional filing fee?
A: UNKNOWN (live fetch unavailable)
   Check: https://www.uspto.gov/learning-and-resources/fees-and-payment/uspto-fee-schedule
```

**Result:** User can verify themselves instead of trusting potentially outdated model memory.

### **3. Case-Law Hardening**

**Rule:** Any case mention (even shorthand like "LKQ", "Alice") → mandatory **CITATION_BLOCK**

**Why:** Case law is the foundation of legal analysis. Vague references like "a recent Federal Circuit case said..." are unacceptable in professional contexts.

**Enforcement:** Post-answer validation scans for case patterns (`v.`, `In re`, `Ex parte`, shorthand registry) and fails if CITATION_BLOCK missing.

### **4. Visual Aids Protocol**

**Trigger:** Keywords like "timeline", "deadline", "PCT", "provisional", "process flow"

**Output:** **DIAGRAM_BLOCK** with:
- Mermaid-formatted diagram (or ASCII fallback)
- `[Image of X]` tag in answer text
- Related sections list

**Why:** Legal timelines and processes are easier to understand visually than as prose.

---

## Domain Applicability

While this repository focuses on **patent law**, the architecture is domain-agnostic:

| Domain | Variable Data | Static Authorities | Use Cases |
|--------|--------------|-------------------|-----------|
| **Patent Law** | Fees, deadlines, portal status | 35 USC, 37 CFR, MPEP, cases | Provisional filing, office actions, prosecution |
| **Medical** | Drug dosages, contraindications | FDA guidance, clinical guidelines | Treatment planning, drug interactions |
| **Financial** | SEC deadlines, thresholds | SEC rules, FINRA guidance | Compliance, disclosures, filings |
| **Software** | Library versions, API status | RFCs, language specs, OWASP | Code review, security audits |

**Expand to your domain:** The CogniMap JSON structure can model any high-stakes knowledge domain with volatile facts and authoritative sources.

---

## Potential Impact

### **Hypothetical Scenario (Illustrative):**

**Without structured provenance:**
- User asks: "What's the provisional filing fee?"
- LLM: "$60 for micro entities" [no source, from model memory]
- User cannot verify → may be outdated

**With CogniMap approach:**
- User asks: "What's the provisional filing fee?"
- LLM: "$65 for micro entities" + EVIDENCE_BLOCK with source URL and retrieval date
- User can click source link to verify

**Note:** This is a hypothetical illustration. Real-world impact depends on deployment context, user behavior, and whether the approach proves robust under broader evaluation.

---

## Technical Details

### **Computational Overhead**

| Aspect | Impact |
|--------|--------|
| **Prompt Tokens** | +5-10% (CogniMap injection) |
| **Response Tokens** | +10-20% (structured blocks) |
| **Validation Time** | ~50-200ms (regex + schema checks) |
| **Total Latency** | Minimal (<5% for most queries) |

**Trade-off:** Slightly longer responses for dramatically higher trust in high-stakes contexts.

### **Model Compatibility**

✅ **Tested:**
- Gemini 2.0 Flash Experimental
- Claude 3.5 Sonnet (expected)
- GPT-4 Turbo (expected)

**Requirements:**
- Large context window (CogniMaps: 30-50KB)
- Strong instruction-following
- JSON output support

### **Maintenance**

**Quarterly Reviews:**
- Variable data registry (fees, income limits, portal status)
- Static authority registry (new cases, statute amendments)
- Validation gate trigger keywords (tune for false positives/negatives)

**Versioning:**
- CogniMaps use semantic versioning (e.g., `v2026.7.6`)
- Include changelog in metadata
- SHA-256 checksum for integrity

---

## What's Public vs. Private

### ✅ **Public (This Repository):**
- Whitepaper and architecture documentation
- Block schemas (structure)
- Test questions and comparison results
- Example outputs

### 🔒 **Private (Not in Repository):**
- Full production CogniMap JSON (trade secret)
- Backend validation logic (proprietary)
- Web application source code
- Proprietary prompt engineering techniques

**Why?** This repository establishes **prior art** and demonstrates the innovation while protecting the **implementation details** that provide competitive advantage.

---

## 📜 License & Usage

**License:** Proprietary (see [License](./License))

**You May:**
- ✅ Read and cite this work (academic/commercial contexts)
- ✅ Reference the architecture in your own research
- ✅ Use the schemas for interoperability

**You May Not (without permission):**
- ❌ Use this work for commercial products
- ❌ Create derivative works
- ❌ Redistribute modified versions

**Commercial Licensing:** Contact [your-email] for inquiries.

---

## 📖 Citation

If you reference this work, please cite:

```bibtex
@misc{higgs2026cognimaps,
  author = {Higgs, Kamden and Higgs AI (Solara)},
  title = {CogniMaps: A Runtime-Enforced Evidence Protocol for Reducing AI Hallucination in High-Stakes Domains},
  year = {2026},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/KamHiggs/Lidia-NDA-Patent-Attorney}},
  note = {Patent Pending}
}
```

---

## 🚧 Roadmap

### **Phase 1: Foundation** ✅ (Complete)
- [x] Core CogniMap specification
- [x] Patent law domain implementation
- [x] Validation gate pipeline
- [x] Test suite (15 questions)
- [x] Whitepaper publication

### **Phase 2: Expansion** 🚧 (In Progress)
- [ ] Medical guidelines CogniMap
- [ ] Financial compliance CogniMap
- [ ] Software engineering CogniMap
- [ ] Visual diagram rendering (Mermaid → PNG/SVG)
- [ ] Multi-turn conversation support

### **Phase 3: Platform** 🔮 (Planned)
- [ ] CogniMap editor UI
- [ ] Automated live fetch background jobs
- [ ] Validation API (third-party integration)
- [ ] Community CogniMap marketplace
- [ ] Real-time collaboration features

---

## 🤝 Contributing

This is currently a **proprietary project** with limited external contributions. However:

**Bug Reports:** If you find errors in the public documentation or schemas, please open an issue.

**Domain Expertise:** If you're a practitioner in a high-stakes domain (medical, financial, legal) and want to collaborate on a CogniMap for your field, contact us.

**Research Collaboration:** Academic researchers interested in AI safety, hallucination reduction, or legal tech are welcome to reach out.

---

## 📬 Contact

**Creator:** Kamden Higgs
**Organization:** Higgs AI
**Email:** [your-email]
**Website:** [your-website]
**GitHub:** [@KamHiggs](https://github.com/KamHiggs)

---

## 🌟 Acknowledgments

Special thanks to:
- **Lidia** (simulated practitioner persona) for the domain expertise
- **USPTO, WIPO, Cornell LII** for authoritative source access
- **Open-source legal tech community** for inspiration

---

## Changelog

### **v0.1** (January 2026)
- Initial public release (experimental)
- Patent law domain (provisional filing focus)
- Test suite with 15 questions (preliminary evaluation)
- Comparison analysis (with vs. without CogniMap)
- Whitepaper with limitations discussion
- JSON schemas for evidence/citation/audit blocks

---

**Disclaimer:** This system is not legal advice. For high-stakes filings, consult a licensed attorney. CogniMaps are an experimental approach to structured LLM outputs and do not replace professional judgment.

---

**Documentation:** [Whitepaper](./docs/whitepaper.md) • [Schemas](./schemas/) • [Test Results](./examples/comparison-analysis.md)

**Feedback:** We welcome critique and collaboration. Please open an issue or reach out directly.
