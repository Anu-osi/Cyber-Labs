# ContextTrust - Claude Code Context File

## What This Project Is

ContextTrust is an AI Knowledge Integrity research system.
It detects poisoned, fake, and manipulated documents in RAG
(Retrieval-Augmented Generation) systems.

Research question: Can we detect poisoned documents through
behavioral signatures WITHOUT knowing which documents are
poisoned in advance?

This is a Master's research project at Melbourne Institute
of Technology. It will become a published paper and open
source tool.

---

## What Is Already Built (V1 - WORKING, DO NOT TOUCH)

Location: All files at root level and src/

### Working Files (V1 - Reference Only)
- app.py → Flask web UI showing comparison results
- src/rag_baseline.py → ChromaDB RAG with NO trust filtering
- src/rag_trust_aware.py → ChromaDB RAG WITH metadata filtering
- src/evaluator.py → Runs 30 questions, computes severity
- src/schemas.py → Pydantic data models
- src/severity_classifier.py → Classifies failures 0-5
- src/corpus_builder.py → Generated the 60 documents
- src/attack_generator.py → Generated attack documents
- src/question_builder.py → Generated test questions
- src/generate_metadata.py → Created doc_metadata.json
- src/metadata.py → Metadata schema

### Working Data (V1)
- corpus/clean/ → 30 approved policy documents
- corpus/attacks/ → 30 poisoned attack documents
- data/questions.json → 30 test questions with expected answers
- data/doc_metadata.json → Trust labels for all documents
- data/attack_registry.json → Attack pack definitions

### V1 Proven Results
- Baseline RAG: retrieves poison 80% of the time
- Trust-aware RAG: reduces poison retrieval to 10%
- This is the baseline we compare everything against

### V1 Critical Finding
Single-dimension trust filtering (approved=True only) is
insufficient. Stale approved documents still act as poison.
Multi-dimensional filter (approved=True AND is_poisoned=False)
is required.

---

## What We Are Building Now (V2)

### Architecture Principle
V1 files stay untouched as reference baseline.
V2 is built as modular components in new directories.
A config-driven pipeline can reproduce V1 OR run new configs.

### V2 Directory Structure Being Built

```
src/
├── embedding/          ← Embedding wrapper (OpenAI + sentence-transformers)
├── chunking/           ← Fixed, semantic, hierarchical chunking
├── indexing/           ← FAISS vector store abstraction
├── retrieval/          ← Semantic, BM25, Hybrid retrievers
├── reranking/          ← Cross-encoder reranker
├── trust/              ← Trust filter (extracted from V1)
├── generation/         ← LLM wrapper
├── answer_gate/        ← Refusal logic for conflicting sources
├── evaluation/         ← Metrics, failure attribution, statistics
├── pipeline/           ← Config-driven system orchestrator
├── behavioral/         ← THE NOVEL CONTRIBUTION (see below)
└── ui/                 ← Streamlit dashboard
experiments/
├── configs/            ← YAML configs for each system variant
└── runs/               ← Saved experiment results
```

### The Novel Research Contribution (Most Important)
Located in: src/behavioral/

This is what makes the project publishable and different from
every other RAG security project.

Six behavioral feature extractors:
1. embedding_outlier.py → Detects docs in unusual vector positions
2. contradiction.py → Cross-document NLI contradiction scoring
3. citation_graph.py → Detects isolated orphan documents
4. temporal_anomaly.py → Detects suspicious sudden policy reversals
5. cooccurrence.py → Detects abnormal retrieval patterns
6. semantic_drift.py → Detects language style anomalies

Plus anomaly classifier that combines all 6 signals without
needing to know which documents are poison in advance.

### What Each V2 Config Will Test
1. v1_baseline.yaml → Reproduce V1 (80% poison) - verification
2. v1_trust.yaml → Reproduce V1 trust-aware (10% poison)
3. v2_hybrid.yaml → BM25 + semantic, no trust
4. v2_hybrid_trust.yaml → Hybrid + trust filtering
5. v2_full.yaml → Hybrid + trust + reranking + answer gate
6. v2_behavioral.yaml → Behavioral detection only
7. v2_combined.yaml → Trust + behavioral combined

---

## Tech Stack

### Current (V1, keep compatible)
- Vector store: ChromaDB (persistent)
- Embeddings: OpenAI ada-002 via ChromaDB default
- LLM: gpt-4o-mini via OpenAI API
- Framework: Flask (UI only)
- Language: Python 3.11

### Adding (V2)
- Additional vector store: FAISS (for scale testing)
- Additional embeddings: sentence-transformers all-MiniLM-L6-v2
- Reranker: cross-encoder/ms-marco-MiniLM-L-6-v2 (HuggingFace)
- BM25: rank-bm25 library
- UI: Streamlit (replaces Flask for research dashboard)
- Experiment tracking: YAML configs + JSON results
- Statistics: scipy, scikit-learn

### Environment
- OS: Windows 11
- Python: 3.11 with venv at ./venv
- Command to activate: venv\Scripts\activate
- Python command: py (not python)
- Package install: py -m pip install

---

## Coding Standards (Mandatory)

- Type hints on all functions
- Docstrings on all public classes and methods
- Every new module has tests in tests/
- No hardcoded values - use config or constants
- Return types must be consistent with existing patterns
- All retrievers return: List[Dict] with keys: chunk_id, text, score, rank, metadata, method
- All evaluators return: Dict with standard result schema

---

## Current Build Status

Phase 1: V2 Foundation (IN PROGRESS)
- [ ] Embedder wrapper
- [ ] Chunking strategies (fixed, semantic)
- [ ] FAISS vector store
- [ ] BM25 retriever
- [ ] Hybrid retriever
- [ ] Cross-encoder reranker
- [ ] Trust filter (extracted from V1)
- [ ] Generator wrapper
- [ ] Evaluation metrics
- [ ] Failure attribution engine
- [ ] Pipeline orchestrator
- [ ] Config system (YAML)
- [ ] Streamlit dashboard

Phase 2: Behavioral Detection (NOT STARTED)
Phase 3: Real-world validation (NOT STARTED)
Phase 4: Adversarial validation (NOT STARTED)
Phase 5: Publication (NOT STARTED)

---

## Rules for Claude Code

1. NEVER modify V1 files (app.py, src/rag_baseline.py,
   src/rag_trust_aware.py, src/evaluator.py)
2. Always follow coding standards above
3. Always run tests after implementing
4. Always commit to git after working state
5. When uncertain about architecture, ask before implementing
6. Use py not python for all commands
7. Activate venv before running anything
8. Check .claude/SESSION_NOTES.md at start of each session
9. Update .claude/SESSION_NOTES.md at end of each session

---

## Key Research Context

This is Master's thesis research, not a portfolio project.
Novel contribution must be scientifically defensible.
All results need statistical significance testing.
Real-world validation on public datasets is required.
Independent adversarial testing is required before publication.

The behavioral detection system must work WITHOUT ground truth
labels - this is the hard constraint and the novel contribution.
