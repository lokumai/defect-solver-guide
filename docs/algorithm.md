# Defect Solver: Algorithm & Methodology

## Core Idea

Traditional bug localization treats this as a **cross-modal retrieval problem**: matching natural language bug reports to source code. This faces three challenges:

1. **Semantic Gap**: Bug descriptions rarely contain exact code terms
2. **Context Limits**: LLMs can't process entire large codebases
3. **Multi-Repository Scale**: No mechanism to first identify the right repository

**Our Solution:** Reframe bug localization as **natural language reasoning** by transforming code into hierarchical summaries, then performing NL-to-NL matching instead of NL-to-code.

```
Traditional Approach:          Our Approach:
Bug Report (NL)               Bug Report (NL)
      ↓                              ↓
  Match to                       Match to
      ↓                              ↓
Source Code                    NL Summaries
                                     ↓
                              Retrieve Code
```

---

## Two-Phase Pipeline

### Phase 1: Knowledge Base Construction (Offline)

Generate hierarchical natural language summaries for each microservice:

```
Repository
    │
    ├─ Project Summary (overview, architecture, purpose)
    │
    ├─ Directory Summaries (module purpose, file relationships)
    │   │
    │   └─ File Summaries (class/method descriptions)
```

**Process:**

1. **File-Level Summarization**
   - Parse each source file
   - Extract class, method, and variable names
   - Generate NL description of file purpose and logic
   - Store: `<repo>_summaries/file_summaries/<file_path>.txt`

2. **Directory-Level Aggregation**
   - Collect all file summaries in a directory
   - Merge and summarize relationships
   - Store: `<repo>_summaries/dir_summaries/<dir_path>.txt`

3. **Repository-Level Summary**
   - Use README, flattened code structure, and directory tree
   - Generate high-level project description
   - Store: `<repo>_summaries/project_summary.txt`

**Example:**
```
product-catalog/
├─ project_summary.txt
│  "Manages product lifecycle, pricing, and inventory..."
│
├─ dir_summaries/
│  ├─ controller.txt
│  │  "REST endpoints for product CRUD operations..."
│  └─ service.txt
│     "Business logic for product validation..."
│
└─ file_summaries/
   ├─ ProductController.java.txt
   │  "Handles HTTP requests for product management..."
   └─ ProductService.java.txt
      "Validates product data and calls repository..."
```

---

### Phase 2: Bug Localization (Online)

#### Step 1: Search Space Routing

**Goal:** Identify top-N microservices likely responsible for the bug.

**Input:** Bug report (summary + description)

**Process:**
1. Load all repository-level summaries
2. Construct prompt: "Given these microservices and this bug, rank them..."
3. LLM ranks microservices by relevance
4. Return top-N candidates (default: 3)

**Prompt Template:**
```
Microservices:
1. product-catalog: Manages product lifecycle, pricing...
2. account: Handles user authentication, profiles...
3. payment: Processes transactions, refunds...
...

Bug Report: "Users cannot update their profile picture"

Rank microservices by likelihood of containing this bug.
```

**Output:**
```json
{
  "selected_spaces": ["account", "user-profile", "document"],
  "scores": [0.85, 0.72, 0.61]
}
```

---

#### Step 2: Bug Localization Within Module

**Goal:** Rank files within a microservice by relevance to the bug.

**Two Strategies:**

##### Strategy A: Direct Ranking (Small Codebases)

Used when all file summaries fit in LLM context.

```
┌─────────────────────────────────────┐
│  Project Summary                    │
│  + All File Summaries               │
│  + Bug Report                       │
│  → LLM ranks files directly         │
└─────────────────────────────────────┘
```

**Prompt:**
```
Project: User account management system...

Files:
1. AuthController.java: Handles login/logout endpoints...
2. ProfileService.java: Updates user profile data...
3. ImageValidator.java: Validates uploaded images...
...

Bug: "Users cannot update profile picture"

Rank top-10 files most likely containing this bug.
```

---

##### Strategy B: Filtered Ranking (Large Codebases)

Used when file summaries exceed LLM context window.

```
Step 1: Directory Filtering
┌─────────────────────────────────────┐
│  Project Summary                    │
│  + All Directory Summaries          │
│  + Bug Report                       │
│  → LLM selects top-N directories    │
└─────────────────────────────────────┘

Step 2: File Ranking (within selected dirs)
┌─────────────────────────────────────┐
│  Project Summary                    │
│  + File Summaries (filtered)        │
│  + Bug Report                       │
│  → LLM ranks files                  │
└─────────────────────────────────────┘
```

**Why Hierarchical?**
- Reduces token usage (only process relevant directories)
- Maintains context (project overview + focused files)
- Improves accuracy (LLM focuses on smaller search space)

---

## Key Design Decisions

### 1. Natural Language as Knowledge Representation

**Why summaries instead of raw code?**

| Aspect | Raw Code | NL Summaries |
|--------|----------|--------------|
| Semantic density | Low (syntax noise) | High (intent focused) |
| Bug-report alignment | Poor (terminology mismatch) | Strong (shared vocabulary) |
| LLM context usage | Inefficient | Efficient |
| Interpretability | Low | High (human-readable path) |

### 2. Hierarchical Structure

**Why repo → directory → file?**

- **Scalability:** Handle codebases exceeding LLM context limits
- **Efficiency:** Filter irrelevant code early
- **Transparency:** Provides clear search path (repo → dir → file)

### 3. Multi-Model Aggregation

For critical tasks, we query multiple LLMs and aggregate:

```
Bug Report → LLM1 → Ranking1
          → LLM2 → Ranking2    →  Weighted Average  →  Final Ranking
          → LLM3 → Ranking3
```

**Benefits:**
- Reduces single-model bias
- Improves robustness
- Configurable per-microservice

---

## Evaluation Results

Tested on **DNext** (46 microservices, 1.1M LOC):

| Method | Pass@10 | MRR |
|--------|---------|-----|
| **Defect Solver (Ours)** | **0.82** | **0.50** |
| GitHub Copilot | 0.34 | 0.19 |
| Cursor | 0.31 | 0.17 |
| BM25 (keyword) | 0.22 | 0.11 |

**Pass@10:** Bug found in top-10 results  
**MRR:** Mean Reciprocal Rank (higher = better ranking)

---

## Example Walkthrough

**Bug Report:**
```
Summary: Payment fails for amounts over $1000
Description: When users try to pay more than $1000, the checkout 
shows "Transaction Error" and logs show "Amount validation failed".
```

**Step 1: Search Space Routing**

Input: Bug report + 46 repository summaries

LLM reasoning:
- "Payment" keyword → payment service likely
- "validation failed" → validation logic issue
- "checkout" → order processing involved

Output: `["payment", "order", "business-validator"]`

---

**Step 2: Bug Localization (payment module)**

Input: Bug report + payment module summaries

Strategy: Direct (43 files fit in context)

File summaries processed:
- `PaymentController.java`: "Handles payment API endpoints..."
- `AmountValidator.java`: "Validates transaction amounts and limits..."
- `TransactionService.java`: "Processes payments via gateway..."

LLM ranking:
1. `validator/AmountValidator.java` (0.92) ← **Bug is here**
2. `service/TransactionService.java` (0.81)
3. `controller/PaymentController.java` (0.73)

Developer investigates `AmountValidator.java` → finds hardcoded $1000 limit.

---

## System Architecture

```
┌────────────────────────────────────────────────────────┐
│                 Offline Phase (Once)                   │
├────────────────────────────────────────────────────────┤
│                                                        │
│  GitHub Repos  →  Summarizer  →  Hugging Face Storage │
│                   (Scheduled)       (Knowledge Base)   │
│                                                        │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│              Online Phase (Per Bug)                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Bug Report → Search Space Router → Selected Repos    │
│                                                        │
│  Selected Repos → Bug Localizer → Ranked Files        │
│                                                        │
│  Ranked Files → Developer → Fix                       │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Advantages Over Baselines

### vs. RAG-based Systems (Copilot, Cursor)

**Limitations of RAG:**
- Retrieves raw code snippets (noisy context)
- No multi-repository reasoning
- Black-box rankings (no search path)

**Our Advantages:**
- Pre-processed NL summaries (clean context)
- Explicit repository routing
- Transparent reasoning (repo → dir → file)

### vs. Keyword Search (BM25)

**Limitations of BM25:**
- Requires exact keyword overlap
- No semantic understanding
- Struggles with paraphrasing

**Our Advantages:**
- LLM semantic matching
- Handles varied terminology
- Captures intent beyond keywords

---

## Limitations & Future Work

### Current Limitations

1. **Static Summaries:** Don't reflect real-time code changes
   - **Mitigation:** Scheduled re-summarization (e.g., every 90 days)

2. **LLM Costs:** Summarization requires API calls
   - **Cost:** ~$80 for 1.1M LOC (one-time)
   - **Future:** Incremental updates, caching strategies

3. **Single Language:** Tested on Java microservices
   - **Extension:** Config supports Python, JavaScript, etc.

### Future Research Directions

- **Incremental Summarization:** Update only changed files
- **Multi-Modal:** Integrate logs, stack traces, metrics
- **Feedback Loop:** Learn from developer corrections
- **Cost Optimization:** Smaller models, selective summarization

---

## Summary

**Problem:** Bug localization in large multi-repository systems fails due to semantic gaps, context limits, and lack of repository routing.

**Solution:** Transform codebases into hierarchical NL summaries and perform NL-to-NL reasoning.

**Results:** 2.4x improvement over state-of-the-art agentic RAG systems.

**Key Insight:** Natural language is a more effective knowledge representation than raw code for LLM-based bug localization.

---

## References

For detailed methodology, evaluation protocol, and ablation studies, see the full research paper in `paper.tex`.
