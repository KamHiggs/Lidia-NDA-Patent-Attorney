# CogniMap Test Questions

**Test Suite Version:** 1.0
**Domain:** US Patent Law & NDAs
**Purpose:** Evaluate LLM output quality with and without CogniMap enforcement

---

## Test Question Set (15 Questions)

These questions are designed to test:
- **Volatile Facts:** Current fees, deadlines, portal status
- **Legal Authorities:** Statutes, regulations, case law
- **Complex Processes:** Timelines, filing procedures
- **Compliance Requirements:** DTSA notices, SEC rules

---

### **Question 1: Current USPTO Fees**
What are the current USPTO provisional filing fees for micro entity, small entity, and standard entity, and what other fees (search/exam) do people mistakenly include when answering this?

**Why This Tests:**
- Volatile facts (fees change regularly)
- Requires EVIDENCE_BLOCK with retrieval proof
- Tests whether system fabricates numbers or forces UNKNOWN

---

### **Question 2: Filing Portal Status**
What is the current USPTO electronic filing system used to submit a new provisional application, and what is the status of EFS-Web today?

**Why This Tests:**
- Infrastructure changes (EFS-Web retired in 2023)
- Tests whether system knows current vs. outdated information
- Requires EVIDENCE_BLOCK for portal status

---

### **Question 3: International Timelines**
What is the correct timeline for international protection: (a) the deadline to file a PCT to claim priority, and (b) the typical deadline for national phase entry?

**Why This Tests:**
- Treaty-based deadlines (relatively stable)
- Tests whether system can cite WIPO sources
- Should trigger DIAGRAM_BLOCK (timeline visualization)

---

### **Question 4: AIA Grace Period**
Under U.S. law, does a public disclosure always create a safe 12-month window to file? Explain the scope and limits of the AIA grace period and why foreign rights can be lost even if the U.S. grace period might apply.

**Why This Tests:**
- Statutory interpretation (35 U.S.C. § 102)
- Requires CITATION_BLOCK for statute
- Tests nuanced understanding of grace period limits

---

### **Question 5: New Matter**
In patent law, what's the practical meaning of "new matter", and how does it affect what you can claim in a non-provisional if your provisional didn't describe a key feature?

**Why This Tests:**
- Legal concept (35 U.S.C. § 112, MPEP guidance)
- Requires CITATION_BLOCK for authorities
- Tests practical understanding of priority support

---

### **Question 6: Enablement vs. Written Description**
Explain the difference between enablement and written description under 35 U.S.C. §112(a), and give an example of how an application can fail one while passing the other.

**Why This Tests:**
- Statutory requirements (35 U.S.C. § 112(a))
- Requires CITATION_BLOCK for statute
- Tests ability to distinguish related concepts

---

### **Question 7: Provisional Claims**
Are formal patent claims required in a provisional application? If not, why do experienced practitioners still include a claim skeleton, and what does it protect against?

**Why This Tests:**
- Regulatory requirements (37 CFR 1.53)
- Requires CITATION_BLOCK for regulation
- Tests field practice vs. legal requirements

---

### **Question 8: GitHub as Prior Art**
Does a public GitHub repository commit count as a patent-defeating disclosure/prior art? What facts determine the answer (public accessibility, timestamps, indexing, etc.)?

**Why This Tests:**
- Prior art rules (35 U.S.C. § 102)
- Requires CITATION_BLOCK for statute
- Tests understanding of "publicly accessible" standard

---

### **Question 9: Micro Entity Status**
What are the eligibility requirements for micro entity status, and what are two common reasons an applicant can accidentally be ineligible?

**Why This Tests:**
- Volatile fact (income limits adjusted annually)
- Requires EVIDENCE_BLOCK for current income limit
- Requires CITATION_BLOCK for eligibility rules (37 CFR 1.29)

---

### **Question 10: SEC Rule 21F-17**
What does SEC Rule 21F-17 prohibit, and does it apply to communications with agencies other than the SEC (e.g., DOJ/OSHA/CFPB/CFTC)?

**Why This Tests:**
- Regulatory interpretation (17 CFR 240.21F-17)
- Requires CITATION_BLOCK for regulation
- Tests understanding of whistleblower protections

---

### **Question 11: DTSA Whistleblower Notice**
What is the DTSA whistleblower immunity notice (18 U.S.C. §1833(b)), when should it appear in NDAs, and what remedies can be impacted if it's missing?

**Why This Tests:**
- Statutory requirement (18 U.S.C. § 1833(b))
- Requires CITATION_BLOCK for statute
- Tests practical compliance knowledge

---

### **Question 12: NDA Confidential Information Definition**
For an NDA, what makes information "confidential" or protectable as a trade secret, and what are the risks of defining confidential information as "anything the company ever says"?

**Why This Tests:**
- Legal definitions (trade secret law)
- Requires CITATION_BLOCK for relevant authorities
- Tests understanding of overbroad clauses

---

### **Question 13: LKQ v. GM (Design Patents)**
What changed in design patent obviousness analysis after LKQ v. GM, and what framework is now used to evaluate obviousness for design patents?

**Why This Tests:**
- **CASE LAW HARDENING:** Must trigger CITATION_BLOCK for LKQ case
- Tests understanding of recent Federal Circuit precedent
- Shorthand "LKQ" should trigger citation requirement

---

### **Question 14: Minimum Provisional Filing Package**
Describe a "minimum safe" US provisional filing package: what documents/components must be included so the submission is correctly treated as a provisional and supports priority (spec, drawings if needed, cover sheet/ADS/identifiers, fee, etc.).

**Why This Tests:**
- Regulatory requirements (37 CFR 1.53)
- Volatile facts (fees)
- Requires both CITATION_BLOCK (regulation) and EVIDENCE_BLOCK (fees)

---

### **Question 15: Scenario - Compromised Foreign Rights**
An inventor publicly demos an invention on January 1, then files a provisional on October 1, and wants protection in the U.S. and abroad. What are the key deadlines for: (a) U.S. non-provisional claiming benefit, (b) PCT filing, and (c) national phase entry—and what rights might already be compromised?

**Why This Tests:**
- Multiple statutory/treaty deadlines
- Requires CITATION_BLOCK for 35 U.S.C. § 119, PCT Articles
- Should trigger DIAGRAM_BLOCK (timeline visualization)
- Tests understanding of absolute novelty vs. grace period

---

## Expected CogniMap Behavior

### ✅ **With CogniMap:**

1. **EVIDENCE_BLOCK** for:
   - Q1: Current provisional fees
   - Q2: Patent Center status / EFS-Web retirement
   - Q9: Micro entity income limit

2. **CITATION_BLOCK** for:
   - Q4: 35 U.S.C. § 102 (grace period)
   - Q5: 35 U.S.C. § 112, MPEP (new matter)
   - Q6: 35 U.S.C. § 112(a) (enablement/written description)
   - Q7: 37 CFR 1.53 (provisional requirements)
   - Q8: 35 U.S.C. § 102 (prior art)
   - Q9: 37 CFR 1.29 (micro entity)
   - Q10: 17 CFR 240.21F-17 (SEC whistleblower)
   - Q11: 18 U.S.C. § 1833(b) (DTSA)
   - Q13: **LKQ Corp. v. GM Global Technology Operations LLC** (case law)
   - Q14: 37 CFR 1.53 (provisional filing)
   - Q15: 35 U.S.C. § 119, PCT Articles (priority)

3. **DIAGRAM_BLOCK** for:
   - Q3: PCT timeline (priority → PCT filing → national phase)
   - Q15: Timeline with disclosure date and filing dates

4. **AUDIT_SUMMARY** with gate pass/fail status

### ❌ **Without CogniMap:**

- Inline footnotes (no structured blocks)
- No explicit EVIDENCE_BLOCK for volatile facts
- No mandatory CITATION_BLOCK for case law
- No DIAGRAM_BLOCK for timelines
- No AUDIT_SUMMARY
- Potentially fabricated or missing retrieval dates

---

## Test Results

See:
- `output-with-cognimap.md` - Full response with CogniMap enforcement
- `output-without-cognimap.md` - Full response without CogniMap
- `comparison-analysis.md` - Side-by-side analysis

---

**Last Updated:** January 2026
