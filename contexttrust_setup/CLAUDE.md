# ContextTrust - Complete Project Context for Claude Code

## Read This Entire File Before Doing Anything

---

## 1. WHAT THIS PROJECT IS

ContextTrust is an AI Knowledge Integrity Framework for RAG systems.

Research question:
Can we detect, prevent, and monitor silent high-risk knowledge
changes in RAG systems before they affect AI-generated answers
and potentially AI-driven actions?

This is simultaneously:
- A Master's research thesis at Melbourne Institute of Technology
- A potential conference/journal paper submission

(See "Future Roadmap" section for longer-term direction beyond
the V3 MVP.)

---

## 2. SECURITY MATURITY MODEL

This framework targets specific security maturity levels.
Do not conflate what we are building now with future levels.

Level 0: No RAG security (documents go straight to vector store)
Level 1: Prompt-only safety (jailbreak protection only)
Level 2: Metadata filtering (V1 and V2 achieved this)
Level 3: Authority-aware retrieval
Level 4: Knowledge staging gate (V3 MVP target)
Level 5: No Silent High-Risk Override (V3 full target)
Level 6: Behavioral and regression monitoring
Level 7: Provenance and workflow integration (SharePoint/Jira/etc)
Level 8: Adaptive red-team testing
Level 9: Enterprise SIEM integration
Level 10: Cryptographic signing, multi-party approval

Current build target: Level 4-5
Thesis claim: Level 4-5
Do NOT claim Level 6+ until built and tested.

---

## 3. WHAT IS ALREADY BUILT

### V1 (COMPLETE - DO NOT MODIFY THESE FILES)

Location: root level and src/

Files (read-only reference):
- app.py
- src/rag_baseline.py
- src/rag_trust_aware.py
- src/evaluator.py
- src/schemas.py
- src/severity_classifier.py
- src/corpus_builder.py
- src/attack_generator.py
- src/question_builder.py
- src/metadata.py

Data (read-only):
- corpus/clean/ (30 approved policy documents)
- corpus/attacks/ (30 poisoned attack documents)
- data/questions.json
- data/doc_metadata.json
- data/attack_registry.json

V1 Status:
Previous runs showed measurable baseline vs trust-aware
differences. Exact results must be cited from saved result
files.

V1 Security Level: Level 2 (metadata filtering)

### V2 (COMPLETE - DO NOT REWRITE OR BREAK)

Status: 189 tests passing, evaluation running

V2 Status:
Previous runs showed measurable baseline vs trust-aware
differences. Exact results must be cited from saved result
files.

V2 Security Level: Level 2-3

V2 Critical Finding:
V2 does not detect unknown/approved-looking poison.
A manually injected document (IT-AUTH-EXC-2026-04) that looked
like a legitimate approved IT policy was retrieved by trust-aware
RAG. The LLM gave a safe answer only because the official policy
was also retrieved and was chosen. This was luck, not security.

V2 proves known poison can be filtered.
V2 does NOT prove unknown poison can be detected.

V2 Files (do not rewrite or break; small integration changes are
allowed if justified, tested, and documented):
- app_v2.py
- src/rag_baseline_v2.py (or equivalent)
- src/rag_trust_aware_v2.py (or equivalent)
- All src/embedding/, src/chunking/, src/retrieval/,
  src/reranking/, src/generation/ modules

---

## 4. WHAT WE ARE BUILDING NOW: V3 MVP

### Core Security Principle

NO SILENT HIGH-RISK OVERRIDE

A new document must not silently change high-impact AI answers
or weaken authoritative rules without detection, review,
and a traceable audit decision.

The system does not need to know absolute truth.
It needs to detect when knowledge changes in dangerous directions.

Dangerous direction patterns (universal, domain-agnostic):
- required → optional
- prohibited → allowed
- approval required → no approval
- verification required → verification delayed
- specific threshold → looser threshold
- mandatory → discretionary
- official only → exceptions accepted

### V3 MVP Scope

The MVP is Layer 1 (Ingestion Gate) plus basic reviewer-feedback
recording. Layer 2 runtime protection, Layer 3 continuous
monitoring, and Layer 4 adaptive thresholding are deferred to
the Future Roadmap.

LAYER 1: INGESTION GATE
Every candidate document passes through before corpus entry.

PLUS: BASIC FALSE-POSITIVE RECORDING
Reviewer decisions are recorded for later analysis. Adaptive
threshold logic and FP-trend dashboards are roadmap items.

### V3 MVP Pipeline

CANDIDATE DOCUMENT ARRIVES
↓
[LAYER 1: INGESTION GATE]
↓
Step 1: STAGING INTAKE
- Assign document ID and timestamp
- Record submitter identity
- Store in corpus/staging/
↓
Step 2: DOCUMENT FINGERPRINTING
- Compute SHA-256 hash of content
- Store in data/fingerprints.json
- On re-ingestion: detect if content changed without new submission
↓
Step 3: INJECTION SCANNER
- Scan for embedded LLM instruction patterns (regex)
- Semantic check: does this text contain AI directives?
- High injection score → HUMAN_REVIEW minimum
↓
Step 4: AUTHORITY CHECK
- Verify submitter against data/trusted_registry.json
- Check submitter is authorized for this document's domain
- Authority fail → HUMAN_REVIEW by default. Hard REJECT only
  if configured.
↓
Step 5: BEHAVIORAL PRE-SCREEN
- Quick anomaly score: contradiction, keyword anomaly,
  authority mismatch, temporal inconsistency
- High behavioural score → HUMAN_REVIEW during research phase
- Otherwise → proceed to full gate
↓
Step 6: CANARY BASELINE RUN (BEFORE state)
- Load data/canary_questions.json (20+ questions)
- Run each question against current trusted corpus
- Use single run for development. Use 3-run median only for
  final evaluation mode. Temperature=0.0
- Store answers as BEFORE state
↓
Step 7: TEMPORARY CORPUS INJECTION
- Add candidate document to in-memory corpus
- Do NOT persist to disk yet
↓
Step 8: CANARY AFTER RUN (AFTER state)
- Run same questions against corpus + candidate
- Same run/temperature policy as Step 6
- Store answers as AFTER state
↓
Step 9: DRIFT DETECTION
- Compare BEFORE vs AFTER per question
- LLM judge determines: changed / direction / severity
- Directions: permissive / restrictive / ambiguous / none
- Only permissive drift on HIGH/CRITICAL questions escalates
↓
Step 10: ENSEMBLE RISK DECISION
- Combine all signals:
  - Authority score
  - Injection score
  - Behavioral score
  - Canary drift score
  - Drift severity
- Agreement rule:
  - 1 signal high → WARN
  - 2 signals high → HUMAN_REVIEW
  - 3+ signals high → REJECT
- Any injection >0.8 → HUMAN_REVIEW minimum
- Authority fail → HUMAN_REVIEW by default (configurable to REJECT)
- Confidence score attached to every decision
- Low confidence (<0.6) → HUMAN_REVIEW regardless of verdict
↓
Step 11: REPORT GENERATION
- Before answers per question
- After answers per question
- Drift detected and direction
- Signals that fired
- Decision and confidence
- Reviewer instructions if HUMAN_REVIEW
- Written to data/gate_reports/[doc_id].json
↓
DECISION:
ALLOW → corpus/clean/, update metadata, update fingerprint
HUMAN_REVIEW → stay in corpus/staging/, alert dashboard
REJECT → corpus/quarantine/, log reason, alert dashboard

(Layer 2 runtime protection, Layer 3 continuous monitoring, and
Layer 4 adaptive thresholding live in section 15 Future Roadmap.)

### V3 MVP Files to Build

```
data/
├── canary_questions.json      ← 20 critical questions (already designed)
├── trusted_registry.json      ← Authorized submitters (already designed)
├── fingerprints.json          ← Document hash registry
├── fp_history.json            ← False positive feedback log
└── gate_reports/              ← Per-document gate decision reports
    └── [doc_id].json

corpus/
├── staging/                   ← Candidate documents waiting
├── quarantine/                ← Rejected documents with evidence
└── archive/                   ← Expired documents (never deleted)

src/knowledge_gate/
├── __init__.py
├── staging.py                 ← Intake, ID assignment, status tracking
├── fingerprint.py             ← SHA-256 hashing, tamper detection
├── injection_scanner.py       ← Regex + semantic injection detection
├── registry.py                ← Authority check against trusted_registry
├── behavioral_screener.py     ← Quick anomaly pre-screen
├── canary_runner.py           ← Run canary questions before/after
├── answer_diff.py             ← Compare answers, detect drift direction
├── drift_classifier.py        ← Label drift: permissive/restrictive/etc
├── ensemble_decision.py       ← Combine signals, compute confidence
├── report.py                  ← Generate human-readable evidence report
└── fp_manager.py              ← False positive recording (basic; adaptive
                                 logic deferred to roadmap)

app_v3.py                      ← New Streamlit entry point with gate UI
experiments/gate_evaluation/
├── attack_scenarios/          ← Test documents for gate testing
├── legitimate_updates/        ← True positives for FP rate measurement
└── results/                   ← Gate decision logs
```

---

## 5. DATA FILES ALREADY DESIGNED

These files have been fully designed. Create them exactly as specified:

### data/canary_questions.json
20 critical questions across domains:
- CQ-IT-001 through CQ-IT-010 (IT Security)
- CQ-HR-001 through CQ-HR-004 (HR Policy)
- CQ-SEC-001 through CQ-SEC-004 (Physical Security)
- CQ-DATA-001 through CQ-DATA-002 (Data Security)

Each question has:
- id, domain, severity (CRITICAL/HIGH/MEDIUM)
- question text
- expected_anchor (semantic anchor, not exact match)
- permissive_drift_pattern
- notes

### data/trusted_registry.json
Three authorized submitters:
- it-security-team (IT Security, IT Access, Data Security)
- hr-team (HR Policy)
- physical-security-team (Physical Security)

### data/fingerprints.json
Initially empty. Populated at ingestion.
Schema: { "doc_id": "sha256_hash" }

### data/fp_history.json
Initially empty. Populated by reviewer decisions.
Schema: list of decision records with override flags.

---

## 6. TECH STACK

### Existing (V1/V2 - do not change)
- Vector store: ChromaDB (persistent)
- Embeddings: OpenAI ada-002 via ChromaDB default
- LLM: gpt-4o-mini via OpenAI API
- Python: 3.11 with venv

### V3 MVP Additions
- LLM judge: gpt-4o-mini temperature=0.0 (same model, separate calls)
- Also test: Claude claude-sonnet-4-6 as alternative judge
- Hashing: hashlib (stdlib, no new dependency)

(APScheduler is in the Future Roadmap once continuous monitoring
is in scope.)

---

## 7. CODING STANDARDS

These are mandatory. Not optional.

- Type hints on every function signature
- Docstrings on every public class and method
- Every new module has corresponding test file in tests/
- No hardcoded values anywhere - use config files or constants
- All retrievers return List[Dict] with standard schema:
  chunk_id, text, score, rank, metadata, method
- All gate decisions return GateDecision dataclass:
  verdict, confidence, signals, explanation, reviewer_needed
- Temperature=0.0 for all LLM judge calls
- Use single run for development. Use 3-run median only for
  final evaluation mode.
- Never auto-REJECT when confidence < 0.6
- Never modify V1 files
- Do not rewrite or break V2. Small integration changes are
  allowed if justified, tested, and documented.

---

## 8. BUILD ORDER

Current task: build V3 MVP gate first.

---

## 9. V3 DASHBOARD REQUIREMENTS

The Streamlit app_v3.py must show these panels:

### Panel 1: Staging Queue
- All documents in staging with status
- Submitter, timestamp, domain, current gate step
- Action buttons: View Report, Approve, Reject

### Panel 2: Gate Decision Detail (per document)
- Before/after answers for each canary question
- Highlight changed answers in amber/red
- Signal scores (authority, injection, behavioral, drift)
- Ensemble confidence score
- Final decision with explanation

### Panel 3: Corpus Health
- Documents by status: active, staging, quarantine, archive
- Documents approaching review_date (amber)
- Documents past expiry_date (red)
(Retrieval amplification alerts deferred to roadmap.)

### Panel 4: Audit Log
- Append-only log of all gate decisions
- Reviewer actions with timestamps
- Corpus changes with before/after states
- Exportable as JSON or CSV

(False Positive Dashboard / FP trend / noisy canary panel deferred
to roadmap.)

---

## 10. WHAT V3 MVP CAN AND CANNOT PROTECT AGAINST

### Protects Against (honest claims only)
- Approved-looking documents that change critical policy answers
- Unknown poison that passes metadata filtering
- Embedded LLM instruction injection
- Documents from unauthorized submitters

### Does NOT Protect Against (state these clearly)
- Compromised approver who approves malicious documents
- Attacks outside canary question coverage
- Slow poisoning within drift threshold per document
- Compromised trusted_registry.json
- Bad LLM judge decisions
- Domain-specific nuance the judge doesn't understand
- False positives on aggressive legitimate policy changes

(Context flooding, answer hallucination, corpus decay, retrieval
amplification, and distributed multi-document drift become in
scope only when the corresponding roadmap layers are built.)

---

## 11. RESEARCH CLAIMS

### Valid claims for thesis/paper
- V3 gate detects policy-weakening documents that V2 metadata
  filtering misses, demonstrated on X attack scenarios
- Ensemble signal agreement reduces false positive rate
  compared to single-signal detection
- No Silent High-Risk Override principle applies across
  IT, HR, security, and data policy domains
- Framework is modular and domain-agnostic

### Invalid claims (never make these)
- ContextTrust secures RAG
- ContextTrust prevents all poisoning
- ContextTrust detects all fake documents
- ContextTrust is production-ready

### Accurate claim
"ContextTrust evaluates and controls silent high-risk knowledge
changes before they affect RAG answers, providing detection and
review for attack classes that metadata filtering cannot address."

---

## 12. ENVIRONMENT

- OS: Windows 11
- Python: 3.11, venv at ./venv
- Activate: venv\Scripts\activate
- Python command: py (not python)
- Install: py -m pip install
- Git: installed and configured
- Repo: cyber-labs on GitHub

---

## 13. SESSION START PROTOCOL

Every session, before writing any code:

1. Read this entire CLAUDE.md
2. Read .claude/SESSION_NOTES.md
3. Summarize current state to confirm understanding
4. State what today's task is
5. Plan the task (show plan, wait for approval)
6. Only then implement

Every session, before ending:

1. Run all tests
2. Commit working state to git
3. Update .claude/SESSION_NOTES.md with:
   - What was completed
   - What is half-finished
   - Open questions
   - Tomorrow's starting point
4. Update build order checkboxes in this file

---

## 14. RULES FOR CLAUDE CODE

1. NEVER modify V1 files
2. Do not rewrite or break V2. Small integration changes are
   allowed if justified, tested, and documented.
3. ALWAYS run tests after each module
4. ALWAYS commit after working state
5. ALWAYS plan before coding
6. NEVER auto-REJECT with confidence < 0.6
7. NEVER hardcode thresholds (use config)
8. ALWAYS use py not python
9. ALWAYS activate venv before running
10. ALWAYS check SESSION_NOTES.md at start
11. NEVER claim the system is secure (it addresses specific threats)
12. ALWAYS update SESSION_NOTES.md at end

---

## 15. FUTURE ROADMAP (out of MVP scope)

These items are deferred until after the V3 MVP is shipped and
evaluated. Do not build them in MVP sessions.

### Long-term product vision
- A RAG Knowledge Security Gateway that becomes the standard
  control plane sitting between enterprise knowledge sources
  (SharePoint, Confluence, Google Drive) and RAG systems
  (LangChain, LlamaIndex, any vector database).
- Build with production-grade modularity from the start.
- Do not shrink this vision. Do not treat the project as a
  student demo.

### Layer 2: Runtime Protection
- src/knowledge_gate/context_budget.py
- src/knowledge_gate/groundedness_checker.py
- Step 12: Context window budget enforcer
  (no single document consumes >35% of context window;
   higher-trust docs get priority budget; truncate lower
   priority chunks before lower trust chunks)
- Step 13: Post-generation groundedness check
  (LLM judge verifies every factual claim is supported by
   retrieved context; ungrounded claims flagged; high
   ungrounded rate logged for investigation)
- Threats addressed: context flooding by large documents,
  answer hallucination beyond retrieved context.

### Layer 3: Continuous Monitoring
- src/knowledge_gate/retrieval_monitor.py
- src/knowledge_gate/decay_monitor.py
- src/knowledge_gate/corpus_regression.py
- Step 14: Retrieval amplification monitor (weekly) — flag
  documents retrieved in >50% of queries across diverse topic
  clusters
- Step 15: Temporal decay monitor (daily) — flag documents past
  review_date; archive past expiry_date; never delete
- Step 16: Corpus-wide canary regression (weekly, Sunday 2am) —
  detect drift caused by combinations of documents
- Dependency: APScheduler
- Threats addressed: retrieval amplification, corpus decay
  from expired documents, distributed multi-document drift.

### Layer 4: Adaptive False-Positive Learning
- src/knowledge_gate/adaptive_thresholds.py
- Step 18: Adaptive threshold adjustment (monthly) — analyse
  last 90 days of decisions; if FP rate >20% relax thresholds
  by ±0.05 bounded; if FP rate <5% tighten
- Step 19: Canary question tuning (monthly) — flag canaries
  with >30% FP rate for administrator review
- 90-day calibration window
- Dashboard Panel: False Positive Dashboard (FP rate, trend,
  noisy canary questions, threshold adjustment history)
- Future valid claim: adaptive thresholding improves gate
  accuracy over a 90-day calibration period.

(`fp_manager.py` basic recording stays in MVP. Only the
adaptive-threshold logic and the FP-trend dashboard panel are
deferred.)
