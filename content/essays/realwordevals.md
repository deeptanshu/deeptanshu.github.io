---
title: "AI Evals: What I learned shipping production grade billing systems"
date: 2026-01-25T13:45:00-08:00
draft: false
category: "RAG"
excerpt: "From Evals 101 to Production Grade systems."
---


## Part 1: Evals 101 - What Are Evaluations?

### The Core Concept

When you build an AI system that answers questions using documents (a RAG system), you need to know: **does it work?**

An evaluation is a systematic way to measure your system's performance using metrics. Instead of manually checking a few examples and saying "looks good," you:

1. Create a test dataset of questions with known correct answers
2. Run your system on those questions
3. Measure how often it gets the right answer
4. Track this score over time

**Example:**

You build a system to answer questions about your company's HR policies.

Manual testing: "Hey, let me try 3 questions. Seems fine!"

Evaluation-based testing:
- Test dataset: 100 HR policy questions
- Your system: 73 correct, 27 incorrect
- Accuracy score: 73%
- After you improve chunking: 84 correct
- New accuracy: 84%
- Confidence: The change worked

### Why This Matters

**Before you have evals:**
- You don't know if changes make things better or worse
- You can't catch regressions (new code breaking old functionality)
- You can't compare different approaches objectively

**After you have evals:**
- Every code change has a score
- You know exactly what improved and what got worse
- You can make data-driven decisions

### The Two-Part System

Every RAG system has two components to evaluate:

**1. Retrieval: Did you find the right documents?**
- Input: User question
- Output: List of relevant documents
- Metric: Did the most relevant document appear in your top results?

**2. Generation: Did you create the right answer?**
- Input: Question + Retrieved documents
- Output: Generated answer
- Metric: Is the answer correct and supported by the documents?

If retrieval fails, generation can't succeed. If you retrieve the wrong documents, even the best language model can't give the right answer.

---

## Part 2: Basic Evaluation Metrics

### Retrieval Metrics

These measure how well your system finds relevant documents.

#### Mean Reciprocal Rank (MRR)

**What it measures:** How quickly users find what they need.

**How it works:**
1. For each question, find the rank (position) of the first relevant document
2. Calculate reciprocal rank = 1 / rank
3. Average across all questions

**Example:**

| Question | Relevant Doc Position | Reciprocal Rank |
|----------|----------------------|-----------------|
| Q1 | 1st result | 1/1 = 1.0 |
| Q2 | 3rd result | 1/3 = 0.33 |
| Q3 | Not in top 10 | 0 |

MRR = (1.0 + 0.33 + 0) / 3 = **0.44**

**What this means:** On average, the first relevant result appears around position 2-3.

**Good score:** MRR > 0.7 means most relevant documents appear in top 2

#### Recall@k

**What it measures:** What percentage of relevant documents appear in your top k results?

**Example:**

Question: "What is our parental leave policy?"
Relevant documents in entire database: 3 documents
Your system returns top 10 results: 2 of those 3 relevant docs are in the top 10

Recall@10 = 2/3 = **0.67** (you found 67% of relevant documents)

**Good score:** Recall@10 > 0.8 means you're finding most relevant information

#### Precision@k

**What it measures:** What percentage of your top k results are actually relevant?

**Example:**

Your system returns top 10 results
6 of those 10 are actually relevant to the question

Precision@10 = 6/10 = **0.6** (60% of what you returned was useful)

**Good score:** Precision@5 > 0.7 means most of what you return is relevant

### Generation Metrics

These measure the quality of the generated answer.

#### Correctness

**What it measures:** Did the answer solve the user's question?

**How to measure:**

Option 1 - Manual: Human reads the answer and marks it correct/incorrect

Option 2 - LLM-as-judge: Use another AI model to grade the answer

```python
def evaluate_answer(question, generated_answer, expected_answer):
    """
    Compare generated answer to expected answer
    Returns: CORRECT, PARTIALLY_CORRECT, or INCORRECT
    """
    # Use an LLM to judge
    prompt = f"""
    Question: {question}
    Expected Answer: {expected_answer}
    Generated Answer: {generated_answer}
    
    Is the generated answer correct?
    Reply with: CORRECT, PARTIALLY_CORRECT, or INCORRECT
    """
    return llm_judge(prompt)
```

#### Faithfulness (Groundedness)

**What it measures:** Is every claim in the answer supported by the retrieved documents?

This catches hallucinations - when the model makes up information not in the source documents.

**Example:**

Retrieved document: "Our company offers 12 weeks of parental leave."

Generated answer: "The company offers 12 weeks of parental leave, which can be extended to 16 weeks with manager approval."

Faithfulness check: ❌ FAILED
- "12 weeks" ✓ supported by document
- "extended to 16 weeks with manager approval" ✗ NOT in document (hallucination)

#### Answer Completeness

**What it measures:** Did the answer include all important information?

**Example:**

Expected answer should contain: [12 weeks leave, applies to all employees, must be taken within first year]

Generated answer mentions: [12 weeks leave, applies to all employees]

Completeness = 2/3 = **0.67** (missing one key fact)

---

## Part 3: Building Your First Evaluation

### Step 1: Create a Test Dataset

Start with 20-30 real questions that users have asked (or will ask).

**Example for HR Policy Q&A:**

```json
[
  {
    "id": "hr_001",
    "question": "How many weeks of parental leave do we offer?",
    "expected_answer": "12 weeks of paid parental leave",
    "relevant_doc_ids": ["policies/parental_leave.pdf"],
    "category": "benefits"
  },
  {
    "id": "hr_002", 
    "question": "What is the remote work policy for engineers?",
    "expected_answer": "Engineers can work remotely up to 3 days per week",
    "relevant_doc_ids": ["policies/remote_work.pdf"],
    "category": "work_arrangements"
  }
  // ... 18 more questions
]
```

### Step 2: Run Your System

For each question in your test dataset:
1. Run retrieval to get documents
2. Generate an answer from those documents
3. Record what happened

```python
results = []
for item in test_dataset:
    # Retrieval
    retrieved_docs = retrieval_system.search(item["question"], top_k=5)
    
    # Generation
    answer = llm.generate(
        question=item["question"],
        context=retrieved_docs
    )
    
    # Record
    results.append({
        "question_id": item["id"],
        "retrieved_docs": retrieved_docs,
        "generated_answer": answer,
        "expected_answer": item["expected_answer"]
    })
```

### Step 3: Calculate Metrics

```python
# Retrieval metrics
def calculate_mrr(results, test_dataset):
    reciprocal_ranks = []
    for result in results:
        # Find position of first relevant doc
        relevant_doc_ids = test_dataset[result["question_id"]]["relevant_doc_ids"]
        
        for position, doc in enumerate(result["retrieved_docs"], start=1):
            if doc.id in relevant_doc_ids:
                reciprocal_ranks.append(1 / position)
                break
        else:
            reciprocal_ranks.append(0)  # No relevant doc found
    
    return sum(reciprocal_ranks) / len(reciprocal_ranks)

# Generation metrics
def calculate_correctness(results):
    correct = 0
    for result in results:
        if llm_judge(result["generated_answer"], result["expected_answer"]) == "CORRECT":
            correct += 1
    
    return correct / len(results)
```

### Step 4: Track Over Time

```python
# Save results
evaluation_run = {
    "timestamp": "2024-01-15",
    "version": "v1.0",
    "metrics": {
        "mrr": 0.65,
        "recall@5": 0.78,
        "precision@5": 0.82,
        "correctness": 0.73,
        "faithfulness": 0.85
    }
}

# Compare to previous version
previous_run = load_previous_run("v0.9")
print(f"MRR improved from {previous_run['mrr']} to {evaluation_run['mrr']}")
```

---

## Part 4: Detailed Case Study - Exa.ai People Search

*Source: https://exa.ai/blog/people-search-benchmark*

### The Problem

Exa built a search engine to find people based on their roles, skills, and locations. For example:
- "Senior software engineer in San Francisco"
- "VP of Product at Figma"
- "Director of sales operations in Chicago SaaS companies"

They indexed 1 billion people from LinkedIn profiles, company websites, and conference speaker bios.

**Challenge:** How do you know if your people search actually works?

You can't manually test 1 billion profiles. You need systematic evaluation.

### Step 1: Understanding User Queries

Exa analyzed 10,000 historical search queries and found 3 patterns:

**Pattern 1: Role-based search** (most common)
- "VP of product at Figma"
- "CTO at Stripe"
- Someone looking for a specific person at a specific company
- **Ground truth:** That person's actual profile exists or doesn't

**Pattern 2: Skill/role discovery**
- "Senior backend engineers in Austin"
- "Marketing directors with B2B SaaS experience"
- Finding anyone who matches multiple criteria
- **Ground truth:** Multiple correct answers, must check if results match ALL criteria

**Pattern 3: Individual lookup**
- "Sarah Chen at Microsoft"
- Name + company affiliation
- **Ground truth:** That specific person's profile

### Step 2: Building the Evaluation Dataset

They created **1,400 test queries** stratified across job functions:

| Job Function | Number of Queries | Examples |
|--------------|-------------------|----------|
| Engineering | 365 | "Senior DevOps engineer", "Security architect" |
| Marketing | 180 | "Growth marketing lead", "Brand manager" |
| Sales | 160 | "Enterprise account executive", "SDR manager" |
| Product | 90 | "Senior product manager", "Head of product" |
| Finance | 85 | "FP&A analyst", "Controller" |
| HR/People | 100 | "People operations manager", "Recruiter" |
| Legal | 70 | "Corporate counsel", "Compliance officer" |
| Design | 100 | "Product designer", "Creative director" |
| Data/Analytics | 70 | "Data scientist", "Analytics engineer" |
| Trust & Safety | 80 | "Trust & Safety specialist", "Policy analyst" |

**Example query structure:**

```json
{
  "query_id": "eng_042",
  "query_text": "Senior software engineer in San Francisco",
  "category": "engineering",
  "query_type": "role_discovery",
  "criteria": {
    "role": "Senior Software Engineer",
    "location": "San Francisco",
    "seniority": "Senior"
  }
}
```

### Step 3: Choosing Evaluation Methods

Exa used **different metrics for different query types:**

**For Pattern 1 & 3 (Specific person lookup):**

Use standard retrieval metrics (Recall@k, MRR, Precision)

```python
# Example evaluation
query = "VP of Product at Figma"
ground_truth_profile = "figma.com/team/jeff-weinstein"  # The actual VP

results = search_engine.search(query, top_k=10)

# Check if correct profile is in results
if ground_truth_profile in results:
    position = results.index(ground_truth_profile) + 1
    reciprocal_rank = 1 / position
else:
    reciprocal_rank = 0
```

**For Pattern 2 (Discovery queries):**

Use **LLM-as-a-judge** because there's no single correct answer.

For each result, check if it matches ALL criteria:

```python
def evaluate_person_result(query_criteria, person_profile):
    """
    Check if person matches all criteria in the query
    Returns: 1 if match, 0 if no match
    """
    prompt = f"""
    Query criteria:
    - Role: {query_criteria['role']}
    - Location: {query_criteria['location']}
    - Seniority: {query_criteria['seniority']}
    
    Person profile:
    {person_profile}
    
    Does this person match ALL criteria?
    Check:
    1. Is their role a match? (e.g., "Software Engineer" matches "Senior Software Engineer")
    2. Is their location a match? (must be exact city)
    3. Is their seniority level correct? (Senior vs Junior vs Staff)
    
    Reply with: MATCH or NO_MATCH
    Explain which criteria failed if NO_MATCH.
    """
    
    result = llm_judge(prompt)
    return 1 if result == "MATCH" else 0
```

**Grading is strict:**

Query: "Senior software engineer in San Francisco"

| Result | Role | Location | Seniority | Score |
|--------|------|----------|-----------|-------|
| Person A | Senior SWE | San Francisco | Senior | ✅ 1 |
| Person B | Senior SWE | San Diego | Senior | ❌ 0 (wrong city) |
| Person C | Junior SWE | San Francisco | Junior | ❌ 0 (wrong level) |
| Person D | Senior PM | San Francisco | Senior | ❌ 0 (wrong role) |

Overall score: 1/4 = 25% precision

### Step 4: Running the Evaluation

Exa tested three search engines:

**Test setup:**
- Same 1,400 queries for all engines
- Each engine returns top 10 results
- Measure: Recall@1 (first result correct), Recall@10 (correct answer in top 10), Precision

**Results:**

| Search Engine | R@1 | R@10 | Precision |
|---------------|-----|------|-----------|
| Exa | 72.0% | 94.5% | 63.3% |
| Brave | 44.4% | 77.9% | 30.2% |
| Parallel | 20.8% | 74.7% | 26.9% |

**What these numbers mean:**

**Exa's R@1 = 72%**
- For 72% of queries, the first result is correct
- User doesn't need to scroll
- Fast, low-friction experience

**Exa's R@10 = 94.5%**
- For 94.5% of queries, the correct answer appears somewhere in top 10
- Answer is findable, even if not first

**Exa's Precision = 63.3%**
- Of all results returned, 63.3% are actually relevant
- Still some noise, but better than competitors

**Compare to Parallel's R@1 = 20.8%**
- Only 1 in 5 queries gets the right first result
- Users scroll through ~5 results on average
- Higher friction, slower

### Step 5: Understanding What Made Exa Better

The 51% improvement in R@1 (from 21% to 72%) came from:

**1. Fine-tuned embeddings**
- Trained specifically on person-query pairs
- "Senior engineer in SF" embeds close to profiles with {role: Senior Engineer, location: SF}
- Not general-purpose text embeddings

**2. Hybrid retrieval**
- Semantic search (embeddings) for fuzzy matching
  - Matches "Software Engineer" to "SWE"
- Keyword search (BM25) for exact terms
  - Ensures "San Francisco" doesn't match "San Jose"
- Combined with Reciprocal Rank Fusion

**3. Profile consolidation**
- Same person appears on LinkedIn, company site, conference speaker page
- System merges these into one profile
- Avoids duplicate results

**4. Fresh data**
- 50 million profile updates per week
- Job changes reflected quickly
- Location updates captured

### Key Takeaways from Exa Case Study

1. **Different query types need different evaluation methods**
   - Factual lookups → standard metrics
   - Open-ended discovery → LLM-as-judge

2. **Strict grading prevents false positives**
   - All criteria must match
   - "Close enough" = wrong

3. **Real user queries matter**
   - 10,000 historical queries → understand actual usage
   - Synthetic queries miss real patterns

4. **Public benchmarks drive progress**
   - Open-sourcing the eval forces competitors to report same metrics
   - Can't cherry-pick easy examples anymore

---

## Part 5: Intermediate Concepts

### LLM-as-a-Judge: Automated Evaluation

For large-scale testing, you can't manually review every answer. LLM-as-a-judge automates this.

**How it works:**

```python
def llm_as_judge(question, answer, expected_answer):
    prompt = f"""
    You are evaluating an AI system's answer.
    
    Question: {question}
    Expected: {expected_answer}
    Actual: {answer}
    
    Rate the actual answer:
    - CORRECT: Fully answers the question
    - PARTIALLY_CORRECT: Right idea but incomplete
    - INCORRECT: Wrong or irrelevant
    
    Provide your rating and a brief explanation.
    """
    
    response = gpt4(prompt)  # or claude, or another strong model
    return parse_rating(response)
```

**Benefits:**
- Scales to thousands of evaluations
- Consistent scoring (doesn't get tired like humans)
- Fast (seconds vs. hours for human review)

**Limitations:**
- Has biases (covered in advanced topics)
- Not perfect - should be calibrated against human judgment
- Best used for regression testing, not absolute measurement

---

> **ADVANCED: LLM-as-Judge Biases**
> 
> LLM judges have systematic biases you should know about:
> 
> **Verbosity Bias:** Judges favor longer answers even when brief answers are better.
> - Solution: Include length guidance in judging prompt
> 
> **Position Bias:** In pairwise comparisons, judges favor the first option.
> - Solution: Randomize order and run twice
> 
> **Circular Logic:** If judge model = generation model, it's biased toward its own outputs.
> - Solution: Use different model family as judge
> 
> **Calibration with Cohen's Kappa:**
> 
> ```python
> # 1. Get human judgments on 100 samples
> human_scores = [evaluate_manually(q) for q in sample_queries[:100]]
> 
> # 2. Get LLM judgments on same samples
> llm_scores = [llm_as_judge(q) for q in sample_queries[:100]]
> 
> # 3. Calculate agreement
> from sklearn.metrics import cohen_kappa_score
> kappa = cohen_kappa_score(human_scores, llm_scores)
> 
> # kappa > 0.75: Good agreement
> # kappa 0.60-0.75: Moderate agreement  
> # kappa < 0.60: Poor agreement, recalibrate judge
> ```

---

### The Precision-Recall Trade-off

You can increase recall (finding more relevant documents) OR increase precision (returning only relevant documents), but it's hard to maximize both.

**Example:**

Your system can retrieve k documents (k = 5, 10, or 20).

| k value | Recall@k | Precision@k | Why |
|---------|----------|-------------|-----|
| k=5 | 0.65 | 0.82 | Few docs, most relevant, but miss some |
| k=10 | 0.83 | 0.71 | More docs, find more, but some noise |
| k=20 | 0.91 | 0.58 | Find almost everything, but half is junk |

**How to decide:**

Consider your use case:
- **High-stakes compliance queries:** Optimize for recall (k=20)
  - Missing information is worse than noise
  - Accept lower precision
  
- **Fast user Q&A:** Optimize for precision (k=5)
  - Users want quick, clean answers
  - Accept that you'll miss some edge cases

**Real-world constraint: Latency**

More documents = more tokens in LLM context = higher latency:
- k=5: ~2,000 tokens, 0.8s generation time
- k=10: ~4,000 tokens, 1.2s generation time  
- k=20: ~8,000 tokens, 2.1s generation time

If your SLA is <1.5s response time, k=20 is not an option.

---

> **ADVANCED: The Pareto Frontier**
>
> You're optimizing three competing objectives:
> 1. Recall (completeness)
> 2. Precision (accuracy)
> 3. Latency (speed)
>
> You cannot maximize all three. This creates a Pareto frontier - the set of optimal trade-offs.
>
> ```
>        High Recall
>             ^
>             |
>        B    |    A (you are here)
>             |
>      C      |
>             |
>         Low Precision -----> High Precision
> ```
>
> - Point A: High precision, medium recall, fast (k=5)
> - Point B: High recall, medium precision, slow (k=20)
> - Point C: Medium on both, medium speed (k=10)
>
> **How production teams decide:**
>
> 1. Set hard constraints
>    - Latency must be < 2.0s (eliminates options above this)
>    - Cost must be < $0.05/query (eliminates expensive configs)
>
> 2. Among remaining options, optimize for primary metric
>    - If primary = user satisfaction → optimize precision
>    - If primary = compliance → optimize recall
>
> From Braintrust research: "RAG systems face strict latency requirements. Users tolerate maybe 2-3 seconds for answers. Retrieval plus generation easily exceeds this budget without optimization."

---

### Building a Golden Dataset

A "golden dataset" is your regression test suite. It's called "golden" because:
- Carefully curated from real failures
- Manually annotated by experts
- Small enough to reason about (50-100 queries)
- Represents your actual query distribution

**How to build one:**

**Step 1: Collect failure cases**

Look for:
- Support tickets that required escalation
- Questions that got wrong answers
- Queries that returned "I don't know" but should have answered
- Edge cases that exposed gaps

**Step 2: Stratify by category**

Example for billing Q&A system:

| Category | Count | Example Query |
|----------|-------|---------------|
| Proration | 15 | "How do we handle mid-cycle upgrades?" |
| Credits | 12 | "Can credits roll over to next month?" |
| Invoicing | 18 | "When are invoices generated?" |
| Taxes | 10 | "How is sales tax calculated?" |
| Edge cases | 20 | "What if customer downgrades same day as upgrade?" |

**Step 3: Annotate thoroughly**

```json
{
  "query_id": "billing_015",
  "query": "How do we handle proration when customer upgrades mid-cycle?",
  "expected_answer": "We use day-based proration. Credit unused days from old plan, charge prorated amount for new plan.",
  "relevant_doc_ids": [
    "docs/proration_logic.md",
    "docs/billing_calculations.md"
  ],
  "category": "proration",
  "difficulty": "medium",
  "requires_calculation": true,
  "should_refuse": false
}
```

**Step 4: Get expert review**

Have domain experts validate:
- Is the expected answer correct?
- Are the relevant docs complete?
- Is the difficulty rating accurate?

---

## Part 6: Continuous Evaluation

### Automated Testing Pipeline

Once you have a golden dataset, integrate it into your development workflow:

```python
# ci_cd_pipeline.py
def run_evaluation_suite():
    """Run before every deployment"""
    
    # Load golden dataset
    golden_dataset = load_golden_dataset()
    
    # Run current system
    results = []
    for query in golden_dataset:
        retrieved = retrieval_system.search(query["question"])
        answer = llm.generate(query["question"], retrieved)
        results.append({
            "query_id": query["id"],
            "answer": answer,
            "retrieved": retrieved
        })
    
    # Calculate metrics
    metrics = {
        "mrr": calculate_mrr(results, golden_dataset),
        "recall@5": calculate_recall_at_k(results, golden_dataset, k=5),
        "correctness": calculate_correctness(results, golden_dataset),
        "faithfulness": calculate_faithfulness(results, golden_dataset)
    }
    
    # Check thresholds
    assert metrics["correctness"] >= 0.85, "Correctness below threshold!"
    assert metrics["faithfulness"] >= 0.90, "Faithfulness below threshold!"
    
    return metrics
```

**When to run:**
- Before every deployment (CI/CD)
- After changing retrieval logic
- After updating document corpus
- When switching LLM models
- Daily as a sanity check

### A/B Testing in Production

Golden datasets don't capture real user behavior. You need production testing.

**Setup:**
```python
# Route 90% to version A, 10% to version B
def handle_query(user_query):
    user_id = get_user_id()
    
    if hash(user_id) % 100 < 90:
        # Version A (current production)
        return version_a.answer(user_query)
    else:
        # Version B (new candidate)
        return version_b.answer(user_query)
```

**Track metrics:**
- User feedback (thumbs up/down)
- Follow-up question rate (did first answer satisfy?)
- Session length
- Task completion rate

**Decision criteria:**

After 2 weeks:
- If Version B's thumbs up rate > Version A's: ramp B to 50%
- Monitor for 1 more week
- If still better: ramp B to 100%
- If worse at any point: rollback to A

---

## Part 7: Advanced Topics

### GraphRAG for Cross-Document Reasoning

> **ADVANCED TOPIC**
>
> **The Problem:**
> 
> Traditional RAG fails on questions like:
> - "What are the main themes across all Q4 board decks?"
> - "How has our security posture evolved over the past year?"
> - "What consistent patterns appear in customer feedback?"
>
> These require synthesizing information across many documents. You can't answer by retrieving 5 chunks.
>
> **GraphRAG Approach:**
>
> 1. **Build knowledge graph**
>    - Extract entities (companies, people, concepts)
>    - Extract relationships between entities
>    - Store as graph: nodes = entities, edges = relationships
>
> 2. **Detect communities**
>    - Find clusters of related entities
>    - Example: All entities related to "product development" form one community
>
> 3. **Generate community summaries**
>    - Each community gets an LLM-generated summary
>    - These summaries become the retrieval units
>
> 4. **Query-time synthesis**
>    - Retrieve relevant community summaries
>    - Synthesize into final answer
>
> **Trade-off:**
> - Much better for global questions
> - 40x more expensive to build index
> - Overkill for simple fact lookup
>
> **When to use:**
> - 10% of queries that need cross-document synthesis
> - Use regular RAG for the other 90%

---

### Drift Detection

> **ADVANCED TOPIC**
>
> Your system's performance can degrade over time even without code changes:
>
> **Behavioral Drift:** Model gets worse
> - Provider updates the model
> - Performance degrades
> - Detection: Track metrics over time, alert if drop >10%
>
> **Concept Drift:** User queries shift
> - Users start asking about "AI agents" 
> - Your docs use term "autonomous systems"
> - Embeddings miss the connection
> - Detection: Track query distribution, alert if significant shift
>
> ```python
> # Weekly drift check
> current_week_queries = get_queries(last_7_days)
> baseline_queries = golden_dataset_queries
> 
> # Cluster and compare distributions
> from scipy.spatial.distance import jensenshannon
> divergence = jensenshannon(
>     cluster_distribution(current_week_queries),
>     cluster_distribution(baseline_queries)
> )
> 
> if divergence > 0.3:
>     alert("Query distribution has shifted significantly")
> ```

---

### Refusal Accuracy

> **ADVANCED TOPIC**
>
> In high-stakes domains, saying "I don't know" when uncertain is safer than hallucinating.
>
> **Metrics:**
>
> **Refusal Precision:** When system refuses, is it correct to refuse?
> - System refused 20 queries
> - 18 were actually unanswerable  
> - Refusal Precision = 18/20 = 90%
>
> **Refusal Recall:** When system should refuse, does it?
> - 25 queries were unanswerable
> - System refused 18 of them
> - Refusal Recall = 18/25 = 72%
>
> The system hallucinated answers for 7 queries it should have refused.
>
> **Implementation:**
>
> ```python
> def generate_answer(query, retrieved_chunks):
>     # Filter low-confidence chunks
>     high_conf = [c for c in retrieved_chunks if c.score > 0.75]
>     
>     if not high_conf:
>         return "I don't have enough information to answer this."
>     
>     if max(c.score for c in retrieved_chunks) < 0.80:
>         return "I'm not confident in the retrieved information."
>     
>     # Proceed with generation
>     return llm.generate(query, high_conf)
> ```

---
