# InfoQ Certified AI Engineering - Quick Reference Guide

**⚡ Last-Minute Exam Prep | Key Concepts & Patterns**

---

## 📋 Exam Preparation Checklist

### Before the Exam
- [ ] Review all 5 weeks of material
- [ ] Complete all self-assessments (score 80%+)
- [ ] Practice with question banks
- [ ] Review capstone project
- [ ] Memorize key patterns and metrics
- [ ] Understand trade-offs for major decisions

### During the Exam
- [ ] Read questions carefully
- [ ] Eliminate wrong answers first
- [ ] Look for keywords (production, scalable, cost-effective)
- [ ] Consider trade-offs in scenario questions
- [ ] Manage time effectively

---

## 🎯 Week 1: AI-Native Engineering

### Key Concepts

**LLM Fundamentals:**
- **Token:** Basic unit of text (word/subword)
- **Context Window:** Max tokens model can process (e.g., 128K, 32K)
- **Temperature:** Controls randomness (0 = deterministic, 1 = creative)
- **Top-P:** Nucleus sampling for diversity control
- **Stop Sequences:** Force model to stop generating

**Context Engineering:**
- **Context Window:** Limited space - optimize carefully
- **Context Rot:** Performance degradation with long contexts
- **Context Compression:** Reduce tokens while preserving meaning
- **Context Window Management:** Critical for production

**Prompt Patterns:**
1. **Zero-Shot:** Direct question, no examples
2. **Few-Shot:** Provide examples in prompt
3. **Chain-of-Thought:** "Think step by step"
4. **System Prompting:** Set behavior at start
5. **Template-Based:** Structured prompts with variables

**Evaluation Metrics:**
- **Perplexity:** How well model predicts text (lower = better)
- **BLEU:** Translation quality (0-1, higher = better)
- **ROUGE:** Summarization quality (precision/recall)
- **BERTScore:** Semantic similarity using embeddings
- **Human Evaluation:** Gold standard but expensive

**Quick Decision Tree:**
```
Need reasoning? → Chain-of-Thought
Need consistency? → Lower temperature
Need creativity? → Higher temperature
Need specific format? → Template-based
Need examples? → Few-shot
```

---

## 🎯 Week 2: RAG & Context Pipelines

### Key Concepts

**RAG Architecture:**
```
Query → Embedding → Vector Search → Context Assembly → LLM → Response
```

**Chunking Strategies:**
- **Fixed-Size:** Simple, predictable (500-1000 tokens)
- **Sentence-Based:** Natural boundaries
- **Paragraph-Based:** Topic coherence
- **Semantic:** Meaning-based boundaries
- **Recursive:** Multi-level splitting

**Context Engineering:**
- **Max Context:** Don't exceed model limits
- **Relevance Ranking:** Most relevant first
- **Context Compression:** Summarize if needed
- **Context Window Management:** Reserve space for response

**Advanced Patterns:**

**HyDE (Hypothetical Document Embeddings):**
1. Generate hypothetical answer
2. Embed hypothetical answer
3. Search with hypothetical embedding
4. Better retrieval for complex queries

**RAG-Fusion:**
1. Generate multiple query variations
2. Search with all variations
3. Combine and rank results
4. Better coverage and recall

**RAG Metrics:**
- **Precision@K:** Relevant docs in top K
- **Recall@K:** % of relevant docs found in top K
- **MRR:** Mean Reciprocal Rank
- **NDCG:** Normalized Discounted Cumulative Gain
- **Answer Accuracy:** LLM-as-judge

**Quick Decision Tree:**
```
Simple retrieval? → Basic RAG
Poor recall? → HyDE or RAG-Fusion
Long documents? → Semantic chunking
Need citations? → Keep source metadata
High latency? → Cache embeddings
```

---

## 🎯 Week 3: AI Agents

### Key Concepts

**Agent Formula:**
```
Agent = LLM + Memory + Tools + Orchestration
```

**Agentic Spectrum:**
1. **Simple Tools:** Function calls, no reasoning
2. **Reasoning Agents:** LLM-based tool selection
3. **Planning Agents:** Multi-step plans
4. **Memory-Enabled:** Short-term + long-term memory
5. **Autonomous:** Self-reflection and learning

**Agent Patterns:**

**ReAct (Reason + Act):**
- Interleave reasoning and action
- Thought → Action → Observation loop
- Best for: Exploratory tasks

**Plan-and-Solve:**
- Create plan first, then execute
- More structured
- Best for: Well-defined tasks

**Reflexion:**
- Self-reflection on failures
- Learn from mistakes
- Best for: Complex, iterative tasks

**Multi-Agent:**
- Multiple specialized agents
- Coordinator assigns tasks
- Best for: Complex tasks requiring expertise

**Memory Types:**
- **Working Memory:** Current task context
- **Episodic Memory:** Past experiences
- **Semantic Memory:** Learned facts
- **Procedural Memory:** Learned procedures

**Safety Mechanisms:**
- **Input Validation:** Block malicious inputs
- **Output Validation:** Filter harmful outputs
- **Rate Limiting:** Prevent abuse
- **Escape Hatches:** Max iterations, time, cost
- **Loop Detection:** Prevent infinite loops

**Failure Modes:**
- **Runaway Loops:** Same action repeated
- **Tool Errors:** API failures, timeouts
- **Goal Drift:** Deviating from original goal
- **Resource Exhaustion:** Running out of tokens/time/money

**Quick Decision Tree:**
```
Simple task? → Single agent
Complex task? → Multi-agent
Need planning? → Plan-and-Solve
Need exploration? → ReAct
Need learning? → Reflexion
High risk? → Add safety guardrails
```

---

## 🎯 Week 4: AI Platforms & Infrastructure

### Key Concepts

**Platform Architecture:**
```
Application Layer (AI Apps)
    ↓
Platform Layer (Gateway, Registry, Feature Store)
    ↓
Infrastructure Layer (Compute, Storage, Network)
```

**Inference Gateway:**
- **Purpose:** Central entry point for model serving
- **Features:** Routing, load balancing, caching, rate limiting
- **Patterns:** Simple proxy, load-balanced, intelligent

**Cost Optimization:**
- **Batching:** Maximize GPU utilization
- **Caching:** Reduce redundant calls
- **Right-sizing:** Match GPU to model
- **Model Selection:** Use smaller models when possible
- **Auto-scaling:** Scale based on demand

**Architecture Decisions:**

**Centralized vs. Federated:**
- **Centralized:** Economies of scale, consistent tooling
- **Federated:** Team autonomy, independent scaling
- **Hybrid:** Best of both (recommended)

**Observability (Three Pillars):**
1. **Metrics:** Latency, throughput, errors
2. **Logs:** Detailed event records
3. **Traces:** Request flow across services

**Key Metrics:**
- **Latency:** p50, p95, p99
- **Throughput:** RPS, TPS
- **Availability:** Uptime percentage
- **Error Rate:** % of failed requests
- **GPU Utilization:** Target 70-80%
- **Cost per Request:** Track and optimize

**Security & Compliance:**
- **Authentication:** API keys, OAuth, mTLS
- **Authorization:** RBAC, model-level permissions
- **PII Handling:** Detect, redact, or block
- **Encryption:** At rest and in transit
- **Audit Logging:** Track all access

**Quick Decision Tree:**
```
High traffic? → Load balancing + caching
Cost concerns? → Batching + right-sizing
Low latency? → Edge deployment + caching
Compliance needed? → PII detection + encryption
Multiple teams? → Hybrid architecture
```

---

## 🎯 Week 5: Operational Excellence

### Key Concepts

**Three-Layer Evaluation:**
1. **Model Layer:** Accuracy, relevance, coherence
2. **System Layer:** Latency, throughput, availability
3. **UX Layer:** CSAT, task completion, time-to-complete

**Evaluation Loop:**
```
Evaluate → Monitor → Collect Feedback → Analyze → Improve → Deploy
```

**Trust Engineering:**
- **Transparency:** Clear about capabilities/limitations
- **Explainability:** Explain decisions
- **Consistency:** Same input → same output
- **Reliability:** Meet SLOs

**SLOs (Service Level Objectives):**
- **Definition:** Target reliability level
- **Examples:** 99.9% availability, <200ms latency
- **Error Budget:** Allowed failure rate (0.1% for 99.9%)

**Rollout Strategies:**

**Canary Deployment:**
- Start with 5% of users
- Monitor metrics
- Gradually increase if successful
- Rollback if issues

**Blue-Green Deployment:**
- Two identical environments
- Switch traffic between them
- Zero-downtime deployments
- Instant rollback

**Rollout Phases:**
1. **Internal Alpha:** Employees only
2. **Canary:** 5% of users
3. **Gradual:** 25% → 50% → 100%
4. **Full Production:** 100% of users

**Key Metrics:**
- **CSAT:** Customer Satisfaction (1-5 scale)
- **NPS:** Net Promoter Score
- **Task Completion Rate:** % of tasks completed
- **Time-to-Complete:** Efficiency metric
- **Escalation Rate:** Human intervention needed

**Quick Decision Tree:**
```
High risk? → Canary deployment
Zero downtime needed? → Blue-green
Need quick rollback? → Blue-green
Gradual rollout? → Phased approach
Trust issues? → Transparency + explainability
```

---

## 📊 Quick Comparison Tables

### RAG Chunking Strategies

| Strategy | Pros | Cons | Best For |
|----------|------|------|----------|
| Fixed-Size | Simple, predictable | May break sentences | General use |
| Sentence | Natural boundaries | Variable size | Conversational text |
| Paragraph | Topic coherence | May be too large | Structured docs |
| Semantic | Meaning-based | Complex | Technical docs |

### Agent Patterns

| Pattern | Complexity | Flexibility | Best For |
|---------|-----------|-------------|----------|
| ReAct | Medium | High | Exploratory tasks |
| Plan-and-Solve | Medium | Low | Well-defined tasks |
| Reflexion | High | High | Complex, iterative |
| Multi-Agent | Very High | Very High | Specialized tasks |

### Deployment Strategies

| Strategy | Risk | Rollback Speed | Complexity |
|----------|------|----------------|------------|
| Big Bang | Very High | Slow | Low |
| Canary | Low | Fast | Medium |
| Blue-Green | Low | Very Fast | High |
| Rolling | Medium | Medium | Medium |

### Architecture Approaches

| Aspect | Centralized | Federated | Hybrid |
|--------|-------------|-----------|--------|
| Cost | Lower | Higher | Medium |
| Flexibility | Low | High | Medium |
| Scalability | Potential bottlenecks | Independent | Best of both |
| Control | High | Low | Medium |

---

## 🧮 Important Formulas & Calculations

### Cost Calculations

**Token Cost:**
```
Cost = (Input Tokens × Input Price + Output Tokens × Output Price) / 1000
```

**GPU Cost Optimization:**
```
Cost per Request = (GPU Cost per Hour × Inference Time) / Batch Size
```

### Evaluation Metrics

**Precision@K:**
```
Precision@K = (# Relevant Docs in Top K) / K
```

**Recall@K:**
```
Recall@K = (# Relevant Docs in Top K) / (Total Relevant Docs)
```

**F1 Score:**
```
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```

**Error Budget:**
```
Error Budget = (1 - SLO Target) × Time Window
Example: 99.9% availability over 30 days = 43.2 minutes downtime
```

### Performance Metrics

**Percentiles:**
- p50: Median (50% of requests faster)
- p95: 95% of requests faster
- p99: 99% of requests faster

**Throughput:**
```
Throughput = Total Requests / Time Period
```

**Availability:**
```
Availability = (Total Time - Downtime) / Total Time
```

---

## 🎓 Key Principles by Week

### Week 1: Context is King
- Context window is limited - optimize carefully
- Prompt engineering dramatically affects output
- Evaluation is multi-dimensional
- LLMs are probabilistic - expect variance

### Week 2: Retrieval Quality Matters
- Garbage in = garbage out
- Chunking strategy affects retrieval
- Context assembly is an art
- Advanced patterns (HyDE, RAG-Fusion) improve recall

### Week 3: Agents Need Guardrails
- Autonomy requires safety mechanisms
- Memory enables learning
- Failure modes are predictable - prevent them
- Choose architecture based on task complexity

### Week 4: Platforms Enable Scale
- Inference costs exceed training costs
- Observability is non-negotiable
- Right architecture depends on organization
- Cost optimization is continuous

### Week 5: Production is Different
- Evaluation spans model, system, UX
- Trust requires multiple factors
- Error budgets enable controlled risk
- Gradual rollout minimizes production risks

---

## 🔑 Must-Know Acronyms

| Acronym | Full Form | Context |
|---------|-----------|---------|
| LLM | Large Language Model | Core technology |
| RAG | Retrieval-Augmented Generation | Architecture pattern |
| API | Application Programming Interface | Integration |
| SLO | Service Level Objective | Reliability target |
| SLA | Service Level Agreement | Contractual commitment |
| UX | User Experience | User satisfaction |
| CSAT | Customer Satisfaction | Metric |
| NPS | Net Promoter Score | Metric |
| PII | Personally Identifiable Information | Privacy |
| RBAC | Role-Based Access Control | Security |
| MLOps | Machine Learning Operations | Practice |
| SRE | Site Reliability Engineering | Practice |
| K8s | Kubernetes | Orchestration |
| GPU | Graphics Processing Unit | Hardware |
| TPU | Tensor Processing Unit | Hardware |
| RPS | Requests Per Second | Metric |
| TPS | Transactions Per Second | Metric |
| MRR | Mean Reciprocal Rank | Evaluation metric |
| NDCG | Normalized Discounted Cumulative Gain | Evaluation metric |

---

## 💡 Common Exam Traps

### Trap 1: "Always use the latest model"
**Reality:** Choose based on cost/performance trade-offs

### Trap 2: "More context is always better"
**Reality:** Context window is limited - optimize carefully

### Trap 3: "Agents are always better than single agents"
**Reality:** Multi-agent adds complexity - use when needed

### Trap 4: "Centralized is always cheaper"
**Reality:** Depends on use case - hybrid often best

### Trap 5: "Model accuracy is the only metric"
**Reality:** Need system and UX metrics too

### Trap 6: "Deploy everything at once"
**Reality:** Gradual rollout minimizes risk

### Trap 7: "Once deployed, work is done"
**Reality:** Continuous evaluation is critical

### Trap 8: "Trust is only about accuracy"
**Reality:** Transparency, explainability, consistency matter too

---

## 📝 Exam Answer Templates

### Architecture Question Template
```
1. Start with requirements (functional + non-functional)
2. Choose architecture pattern (monolithic/microservices)
3. Justify component selection
4. Address scalability, reliability, security
5. Discuss trade-offs
6. Provide implementation roadmap
```

### Trade-off Analysis Template
```
Option A:
- Pros: [list]
- Cons: [list]
- Best for: [scenarios]

Option B:
- Pros: [list]
- Cons: [list]
- Best for: [scenarios]

Recommendation: [choice] because [rationale]
```

### Evaluation Framework Template
```
Model Layer:
- Metrics: [list]
- Baseline: [current]
- Target: [goal]

System Layer:
- Metrics: [list]
- Baseline: [current]
- Target: [goal]

UX Layer:
- Metrics: [list]
- Baseline: [current]
- Target: [goal]

Evaluation Loop: [describe frequency and process]
```

---

## ⚡ 60-Second Summaries

### Week 1: AI-Native Engineering
LLMs are probabilistic text generators. Context engineering (prompt design) dramatically affects output. Evaluate using multiple metrics (perplexity, BLEU, human eval). Production requires careful context window management.

### Week 2: RAG & Context Pipelines
RAG combines retrieval with generation. Chunking strategy affects quality. Context engineering (assembly, compression) is critical. Advanced patterns (HyDE, RAG-Fusion) improve recall. Evaluate with precision/recall metrics.

### Week 3: AI Agents
Agents = LLM + Memory + Tools + Orchestration. Choose architecture based on task complexity. Safety guardrails are non-negotiable. Prevent failure modes (loops, drift, resource exhaustion). Multi-agent for complex tasks.

### Week 4: AI Platforms & Infrastructure
Inference costs exceed training costs. Design for observability (metrics, logs, traces). Choose architecture (centralized/federated/hybrid) based on needs. Optimize costs (batching, caching, right-sizing). Security and compliance built-in.

### Week 5: Operational Excellence
Evaluate across three layers (model, system, UX). Build trust (transparency, explainability, consistency). Define SLOs and error budgets. Gradual rollout (canary → blue-green). Continuous evaluation is non-negotiable.

---

## 🎯 Final Tips

### Day Before Exam
- [ ] Review this quick reference guide
- [ ] Skim through weekly summaries
- [ ] Review practice questions you got wrong
- [ ] Get good sleep
- [ ] Prepare your environment

### During Exam
- [ ] Read questions twice
- [ ] Eliminate obviously wrong answers
- [ ] Look for keywords (production, scalable, cost-effective)
- [ ] Consider trade-offs
- [ ] Manage time (don't get stuck)
- [ ] Review answers if time permits

### Mindset
- ✅ Think like an engineer (practical, not theoretical)
- ✅ Consider trade-offs (no perfect solutions)
- ✅ Focus on production readiness
- ✅ Prioritize reliability and safety
- ✅ Optimize for cost and performance

---

## 📚 Quick Links to Study Materials

| Topic | File | Key Sections |
|-------|------|--------------|
| LLM Fundamentals | Week-01 | Context Engineering, Prompt Patterns |
| RAG Architecture | Week-02 | Chunking, Context Engineering, Advanced Patterns |
| Agent Design | Week-03 | Agentic Spectrum, Patterns, Safety |
| Platform Design | Week-04 | Inference Gateway, Cost Optimization, Observability |
| Operational Excellence | Week-05 | Evaluation Frameworks, Trust, Rollout Strategies |

---

## 🏆 You've Got This!

**Remember:**
- You've completed 50-60 hours of study
- You've practiced with 125+ questions
- You understand the concepts deeply
- You can design production-ready AI systems

**Confidence is key - trust your preparation! 🚀**

---

*Good luck on your certification exam!* 🎓

*Last Updated: 2026*