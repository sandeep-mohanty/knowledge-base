# Week 2: Designing & Building RAG & Context Pipelines - Complete Guide

**📅 Week:** 2 of 5  
**⏱️ Estimated Time:** 10-12 hours  
**🎯 Difficulty:** Intermediate to Advanced  
**📝 Type:** Core Technical Deep Dive

---

## Table of Contents

1. [Introduction](#introduction)
2. [RAG Fundamentals](#rag-fundamentals)
3. [RAG Architecture Deep Dive](#rag-architecture-deep-dive)
4. [Vector RAG vs Graph RAG](#vector-rag-vs-graph-rag)
5. [Context Engineering Principles](#context-engineering-principles)
6. [Memory Systems Design](#memory-systems-design)
7. [Knowledge Graph Deployment](#knowledge-graph-deployment)
8. [Production-Grade Retrieval Design](#production-grade-retrieval-design)
9. [Handling Data Freshness & Changes](#handling-data-freshness--changes)
10. [Hands-On Exercises](#hands-on-exercises)
11. [Practice Question Bank](#practice-question-bank)
12. [Self-Assessment Checklist](#self-assessment-checklist)
13. [Summary & Key Takeaways](#summary--key-takeaways)
14. [Further Reading](#further-reading)

---

## Introduction

Welcome to Week 2 of the InfoQ Certified AI Engineering Program. This week focuses on **RAG (Retrieval-Augmented Generation)** and **Context Pipelines** - the workhorse patterns of enterprise AI systems.

### Learning Objectives

By the end of this week, you will be able to:

✅ **Design** production-grade RAG architectures from scratch  
✅ **Compare** vector RAG vs. Graph RAG and choose the right approach for different use cases  
✅ **Implement** context engineering pipelines that separate ephemeral from durable knowledge  
✅ **Design** memory systems (ephemeral vs. long-term) for AI applications  
✅ **Deploy** knowledge graphs for enhanced reasoning and retrieval  
✅ **Handle** data freshness, changes, and drift in production RAG systems  
✅ **Optimize** retrieval quality through chunking strategies, embedding selection, and reranking  
✅ **Identify** where traditional RAG fails and apply advanced patterns to fix it  

### Why RAG Matters

> 💡 **The RAG Imperative:** LLMs are powerful but limited by their training data. RAG bridges this gap by grounding AI responses in your organization's specific knowledge, making AI systems accurate, current, and trustworthy.

**Key Statistics:**
- **RAG reduces hallucinations by 60-80%** compared to pure LLM approaches
- **Enterprise AI projects using RAG are 3x more likely to succeed** than those relying solely on fine-tuning
- **RAG enables real-time knowledge updates** without retraining models
- **Cost-effective:** RAG with retrieval is 10-100x cheaper than fine-tuning large models

---

## RAG Fundamentals

### What is RAG?

**Retrieval-Augmented Generation (RAG)** is an AI architecture that enhances LLM responses by retrieving relevant information from external knowledge sources before generating answers.

**The RAG Formula:**
```
Response = LLM(Query + Retrieved_Context)
```

### The Problem RAG Solves

#### Problem 1: Knowledge Cutoff
LLMs have a fixed knowledge cutoff. RAG provides access to current, organization-specific information.

#### Problem 2: Hallucinations
RAG grounds responses in retrieved documents, reducing hallucinations by 60-80%.

#### Problem 3: Cost and Latency
RAG is 10-100x cheaper than fine-tuning and enables real-time knowledge updates.

---

## RAG Architecture Deep Dive

### Core Components

#### 1. Document Ingestion Pipeline

```python
class DocumentIngestionPipeline:
    """Ingest documents into RAG knowledge base"""
    def __init__(self, chunker, embedder, vector_store):
        self.chunker = chunker
        self.embedder = embedder
        self.vector_store = vector_store
    
    def ingest_document(self, document):
        """Process and store document"""
        # Step 1: Load document
        raw_text = self.load_document(document)
        metadata = self.extract_metadata(document)
        
        # Step 2: Chunk document
        chunks = self.chunker.chunk(raw_text)
        
        # Step 3: Generate embeddings
        embeddings = [self.embedder.embed(chunk.text) for chunk in chunks]
        
        # Step 4: Store in vector database
        for i, (chunk, embedding) in enumerate(zip(chunks, embeddings)):
            self.vector_store.add(
                id=f"{document.id}_chunk_{i}",
                vector=embedding,
                metadata={
                    'document_id': document.id,
                    'chunk_index': i,
                    'text': chunk.text,
                    'source': document.source,
                    'timestamp': datetime.now().isoformat(),
                    **metadata
                }
            )
        
        return {'document_id': document.id, 'chunks_created': len(chunks)}
```

#### 2. Query Processing

```python
class QueryProcessor:
    """Process and optimize queries for retrieval"""
    def __init__(self, llm):
        self.llm = llm
    
    def process_query(self, query, conversation_history=None):
        """Process query through multiple stages"""
        # Step 1: Query understanding
        intent = self.classify_intent(query)
        entities = self.extract_entities(query)
        
        # Step 2: Query expansion
        expanded_queries = self.expand_query(query)
        
        # Step 3: Query rewriting (if needed)
        if self.needs_rewrite(query, conversation_history):
            rewritten_query = self.rewrite_query(query, conversation_history)
        else:
            rewritten_query = query
        
        # Step 4: Generate embedding
        query_embedding = self.embed_query(rewritten_query)
        
        return {
            'original_query': query,
            'processed_query': rewritten_query,
            'intent': intent,
            'entities': entities,
            'expanded_queries': expanded_queries,
            'embedding': query_embedding
        }
```

#### 3. Retrieval Engine

```python
class RetrievalEngine:
    """Core retrieval logic for RAG"""
    def __init__(self, vector_store, reranker, top_k=10):
        self.vector_store = vector_store
        self.reranker = reranker
        self.top_k = top_k
    
    def retrieve(self, query_embedding, filters=None, top_k=None):
        """Retrieve relevant documents"""
        top_k = top_k or self.top_k
        
        # Step 1: Initial vector search
        initial_results = self.vector_store.search(
            vector=query_embedding,
            top_k=top_k * 3,  # Retrieve more for reranking
            filters=filters
        )
        
        # Step 2: Rerank results
        reranked_results = self.reranker.rerank(
            query=query_embedding,
            documents=initial_results,
            top_k=top_k
        )
        
        # Step 3: Filter by relevance threshold
        filtered_results = [doc for doc in reranked_results if doc['score'] > 0.7]
        
        return filtered_results
```

#### 4. Context Assembly

```python
class ContextAssembler:
    """Assemble retrieved documents into context for LLM"""
    def __init__(self, max_tokens=4000):
        self.max_tokens = max_tokens
        self.tokenizer = Tokenizer()
    
    def assemble_context(self, query, retrieved_docs, max_docs=5):
        """Assemble context from retrieved documents"""
        selected_docs = retrieved_docs[:max_docs]
        context_parts = []
        total_tokens = 0
        
        for i, doc in enumerate(selected_docs):
            doc_text = self.format_document(doc, i + 1)
            doc_tokens = self.tokenizer.count_tokens(doc_text)
            
            if total_tokens + doc_tokens > self.max_tokens:
                break
            
            context_parts.append(doc_text)
            total_tokens += doc_tokens
        
        context = "\n\n".join(context_parts)
        
        return {
            'context': context,
            'documents_used': len(context_parts),
            'total_tokens': total_tokens,
            'sources': [doc['metadata']['source'] for doc in selected_docs[:len(context_parts)]]
        }
    
    def format_document(self, doc, number):
        """Format document with metadata"""
        source = doc['metadata'].get('source', 'Unknown')
        timestamp = doc['metadata'].get('timestamp', '')
        
        return f"""[Document {number}]
Source: {source}
{f"Last Updated: {timestamp}" if timestamp else ""}

{doc['text']}
"""
```

### Complete RAG System

```python
class RAGSystem:
    """Complete RAG system integrating all components"""
    def __init__(self, config):
        self.ingestion_pipeline = DocumentIngestionPipeline(
            chunker=config['chunker'],
            embedder=config['embedder'],
            vector_store=config['vector_store']
        )
        
        self.query_processor = QueryProcessor(llm=config['llm'])
        self.retrieval_engine = RetrievalEngine(
            vector_store=config['vector_store'],
            reranker=config['reranker']
        )
        self.context_assembler = ContextAssembler(
            max_tokens=config.get('max_context_tokens', 4000)
        )
    
    def query(self, user_query, conversation_history=None, filters=None):
        """Process user query through RAG pipeline"""
        # Step 1: Process query
        processed_query = self.query_processor.process_query(user_query, conversation_history)
        
        # Step 2: Retrieve relevant documents
        retrieved_docs = self.retrieval_engine.retrieve(
            query_embedding=processed_query['embedding'],
            filters=filters
        )
        
        # Step 3: Assemble context
        context_result = self.context_assembler.assemble_context(user_query, retrieved_docs)
        
        # Step 4: Generate response
        prompt = self.context_assembler.build_prompt(user_query, context_result['context'])
        response = self.llm.generate(prompt)
        
        return {
            'query': user_query,
            'answer': response,
            'sources': context_result['sources'],
            'confidence': self.calculate_confidence(retrieved_docs)
        }
```

---

## Vector RAG vs Graph RAG

### Vector RAG Deep Dive

Vector RAG uses embeddings and similarity search to retrieve documents.

```python
class VectorRAG:
    """Vector-based RAG implementation"""
    def __init__(self, vector_store, embedder, top_k=5):
        self.vector_store = vector_store
        self.embedder = embedder
        self.top_k = top_k
    
    def retrieve(self, query):
        """Retrieve using vector similarity"""
        query_embedding = self.embedder.embed(query)
        results = self.vector_store.search(vector=query_embedding, top_k=self.top_k)
        return results
```

**Strengths:**
- ✅ Fast and scalable
- ✅ Simple to implement
- ✅ Works well for straightforward queries
- ✅ Mature tooling

**Weaknesses:**
- ❌ No understanding of relationships
- ❌ Struggles with multi-hop reasoning
- ❌ Can miss connected context

### Graph RAG Deep Dive

Graph RAG uses knowledge graphs to capture relationships between entities.

```python
class GraphRAG:
    """Graph-based RAG implementation"""
    def __init__(self, knowledge_graph, llm):
        self.kg = knowledge_graph
        self.llm = llm
    
    def retrieve(self, query):
        """Retrieve using graph traversal"""
        # Step 1: Extract entities from query
        entities = self.extract_entities(query)
        
        # Step 2: Find relevant nodes and traverse
        subgraphs = []
        for entity in entities:
            node = self.kg.find_node(entity)
            if node:
                subgraph = self.kg.get_subgraph(node, max_hops=2)
                subgraphs.append(subgraph)
        
        # Step 3: Convert to text
        contexts = [self.graph_to_text(sg) for sg in subgraphs]
        return contexts
```

**Strengths:**
- ✅ Captures relationships and connections
- ✅ Excels at multi-hop reasoning
- ✅ Explainable reasoning paths
- ✅ Handles complex queries

**Weaknesses:**
- ❌ More complex to build and maintain
- ❌ Slower than vector search
- ❌ Requires graph database

### Decision Framework

| Aspect | Vector RAG | Graph RAG | Hybrid |
|--------|-----------|-----------|--------|
| **Setup Complexity** | Low | High | Medium-High |
| **Query Speed** | Fast (ms) | Slower (100ms-1s) | Medium |
| **Multi-hop Reasoning** | Poor | Excellent | Excellent |
| **Best For** | Simple Q&A | Complex relationships | Both |

**When to use what:**
- **Vector RAG:** Simple factual queries, high-volume search
- **Graph RAG:** Multi-hop reasoning, complex relationships, explainable AI
- **Hybrid:** Enterprise knowledge bases requiring both approaches

---

## Context Engineering Principles

### What is Context Engineering?

**Context Engineering** is the discipline of designing, managing, and optimizing the information provided to LLMs to produce optimal outputs.

### Context Window Challenge

LLMs have limited context windows (8K-128K tokens). Every token counts.

### Key Principles

#### 1. Relevance Over Completeness
Include only what's needed, not everything "just in case."

#### 2. Structure for Comprehension
Organize context hierarchically to help LLM understand relationships.

```python
class StructuredContextBuilder:
    def build_context(self, query, documents, entities):
        context_parts = []
        
        # Level 1: Most relevant documents
        primary_docs = self.get_primary_documents(documents, top_k=3)
        context_parts.append("## Most Relevant Information\n")
        for doc in primary_docs:
            context_parts.append(self.format_document(doc))
        
        # Level 2: Supporting information
        supporting_docs = self.get_supporting_documents(documents, skip=3, top_k=3)
        if supporting_docs:
            context_parts.append("\n## Supporting Information\n")
            for doc in supporting_docs:
                context_parts.append(self.format_document(doc))
        
        return "\n".join(context_parts)
```

#### 3. Progressive Disclosure
Start with summaries, provide details on demand.

#### 4. Context Separation
Separate different context types to avoid confusion.

```python
class ContextSeparator:
    def __init__(self):
        self.contexts = {
            'system': [],      # System instructions
            'history': [],     # Conversation history
            'retrieved': [],   # Retrieved documents
            'user': []         # User profile
        }
    
    def build_prompt(self, query):
        prompt_parts = []
        
        if self.contexts['system']:
            prompt_parts.append("## System Instructions\n" + "\n".join(self.contexts['system']))
        
        if self.contexts['retrieved']:
            prompt_parts.append("\n## Knowledge Base\n" + "\n".join(self.contexts['retrieved']))
        
        if self.contexts['user']:
            prompt_parts.append("\n## User Profile\n" + "\n".join(self.contexts['user']))
        
        prompt_parts.append(f"\n## Current Question\n{query}\n")
        
        return "\n".join(prompt_parts)
```

### Context Optimization Techniques

#### Technique 1: Context Compression

```python
class ContextCompressor:
    """Compress context to fit within token limits"""
    def __init__(self, llm):
        self.llm = llm
    
    def compress_documents(self, documents, max_tokens=2000):
        """Compress documents while preserving key information"""
        compressed = []
        total_tokens = 0
        
        for doc in documents:
            doc_tokens = self.count_tokens(doc['text'])
            
            if total_tokens + doc_tokens <= max_tokens:
                compressed.append(doc)
                total_tokens += doc_tokens
            else:
                remaining_tokens = max_tokens - total_tokens
                if remaining_tokens > 100:
                    compressed_text = self.summarize(doc['text'], remaining_tokens)
                    compressed.append({**doc, 'text': compressed_text, 'compressed': True})
                break
        
        return compressed
```

#### Technique 2: Context Caching

```python
class ContextCache:
    """Cache frequently accessed context"""
    def __init__(self, ttl=3600):
        self.cache = {}
        self.ttl = ttl
    
    def get_or_compute_context(self, query, compute_fn):
        """Get cached context or compute new one"""
        query_hash = self.hash_query(query)
        
        cached = self.get_cached_context(query_hash)
        if cached:
            return cached
        
        context = compute_fn(query)
        self.cache_context(query_hash, context)
        
        return context
```

---

## Memory Systems Design

### Types of Memory

```
┌──────────────────────────────────────────────────────────┐
│              AI System Memory Architecture                 │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Working Memory (Ephemeral)                              │
│  • Current conversation context                          │
│  • TTL: Minutes to hours                                 │
│                                                          │
│  Short-Term Memory (Session)                             │
│  • User session data                                     │
│  • TTL: Hours to days                                    │
│                                                          │
│  Long-Term Memory (Persistent)                           │
│  • User preferences, learned facts                       │
│  • TTL: Permanent                                        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Ephemeral Memory

```python
class EphemeralMemory:
    """Temporary memory for single conversation"""
    def __init__(self, max_turns=10):
        self.max_turns = max_turns
        self.conversation_history = []
    
    def add_message(self, role, content):
        """Add message to conversation history"""
        self.conversation_history.append({
            'role': role,
            'content': content,
            'timestamp': datetime.now()
        })
        
        # Keep only recent turns
        if len(self.conversation_history) > self.max_turns * 2:
            self.conversation_history = self.conversation_history[-self.max_turns * 2:]
    
    def get_context(self):
        """Get current conversation context"""
        return {
            'history': self.conversation_history,
            'turn_count': len(self.conversation_history) // 2
        }
```

### Session Memory

```python
class SessionMemory:
    """Session-based memory for user interactions"""
    def __init__(self, redis_client, session_ttl=86400):
        self.redis = redis_client
        self.session_ttl = session_ttl  # 24 hours
    
    def create_session(self, user_id):
        """Create new session"""
        session_id = generate_session_id()
        session_data = {
            'user_id': user_id,
            'created_at': datetime.now().isoformat(),
            'interactions': [],
            'preferences': {}
        }
        
        self.redis.setex(f"session:{session_id}", self.session_ttl, json.dumps(session_data))
        return session_id
    
    def add_interaction(self, session_id, interaction):
        """Add interaction to session"""
        session = self.get_session(session_id)
        if session:
            session['interactions'].append({**interaction, 'timestamp': datetime.now().isoformat()})
            self.redis.setex(f"session:{session_id}", self.session_ttl, json.dumps(session))
```

### Long-Term Memory

```python
class LongTermMemory:
    """Persistent memory for user data"""
    def __init__(self, database):
        self.db = database
    
    def store_fact(self, user_id, fact_key, fact_value, confidence=1.0):
        """Store a fact about user"""
        fact = {
            'user_id': user_id,
            'key': fact_key,
            'value': fact_value,
            'confidence': confidence,
            'learned_at': datetime.now().isoformat(),
            'last_accessed': datetime.now().isoformat(),
            'access_count': 0
        }
        
        self.db.upsert('user_facts', fact, unique_keys=['user_id', 'key'])
    
    def retrieve_all_facts(self, user_id):
        """Retrieve all facts about user"""
        facts = self.db.query('user_facts', user_id=user_id)
        facts.sort(key=lambda x: (x['confidence'], x['last_accessed']), reverse=True)
        return {fact['key']: fact['value'] for fact in facts}
    
    def learn_from_interaction(self, user_id, interaction):
        """Learn facts from user interaction"""
        extracted_facts = self.extract_facts(interaction)
        
        for fact in extracted_facts:
            existing = self.retrieve_fact(user_id, fact['key'])
            
            if existing:
                confidence = min(1.0, fact['confidence'] + 0.1) if existing == fact['value'] else fact['confidence'] * 0.5
            else:
                confidence = fact['confidence']
            
            self.store_fact(user_id, fact['key'], fact['value'], confidence)
```

---

## Knowledge Graph Deployment

### What is a Knowledge Graph?

A **Knowledge Graph** is a structured representation of knowledge that captures entities (nodes) and their relationships (edges).

### Building a Knowledge Graph

#### Step 1: Entity Extraction

```python
class EntityExtractor:
    """Extract entities from text"""
    def __init__(self, llm):
        self.llm = llm
    
    def extract_entities(self, text):
        """Extract entities from text"""
        prompt = f"""
        Extract all entities (people, organizations, technologies, concepts) from:
        {text}
        
        Return as JSON: {{"entities": [{{"name": "entity_name", "type": "PERSON/ORG/TECH/CONCEPT"}}]}}
        """
        response = self.llm.generate(prompt)
        return json.loads(response)['entities']
```

#### Step 2: Relationship Extraction

```python
class RelationshipExtractor:
    """Extract relationships between entities"""
    def __init__(self, llm):
        self.llm = llm
    
    def extract_relationships(self, text, entities):
        """Extract relationships between entities"""
        entity_names = [e['name'] for e in entities]
        
        prompt = f"""
        Extract relationships between these entities in the text:
        Entities: {', '.join(entity_names)}
        
        Text: {text}
        
        Return as JSON: {{"relationships": [{{"source": "e1", "target": "e2", "type": "WORKS_AT"}}]}}
        """
        response = self.llm.generate(prompt)
        return json.loads(response)['relationships']
```

#### Step 3: Graph Construction

```python
class KnowledgeGraphBuilder:
    """Build knowledge graph from extracted data"""
    def __init__(self, graph_db):
        self.graph_db = graph_db
    
    def build_graph(self, documents):
        """Build knowledge graph from documents"""
        entity_extractor = EntityExtractor(self.llm)
        rel_extractor = RelationshipExtractor(self.llm)
        
        for doc in documents:
            entities = entity_extractor.extract_entities(doc['text'])
            
            for entity in entities:
                self.graph_db.add_node(id=entity['name'], labels=[entity['type']])
            
            relationships = rel_extractor.extract_relationships(doc['text'], entities)
            
            for rel in relationships:
                self.graph_db.add_edge(
                    source=rel['source'],
                    target=rel['target'],
                    relationship_type=rel['type']
                )
```

### Graph Database Options

| Database | Type | Best For | Cost | Complexity |
|----------|------|----------|------|------------|
| **Neo4j** | Native graph | Complex queries | Medium | Medium |
| **Amazon Neptune** | Managed graph | AWS ecosystem | High | Low |
| **JanusGraph** | Distributed | Large-scale graphs | Low | High |
| **NetworkX** | In-memory | Prototyping | Free | Low |

---

## Production-Grade Retrieval Design

### Hybrid Search: Vector + Keyword

```python
class HybridSearchEngine:
    """Hybrid search combining vector and keyword search"""
    def __init__(self, vector_store, keyword_index, alpha=0.5):
        self.vector_store = vector_store
        self.keyword_index = keyword_index
        self.alpha = alpha  # Weight for vector search
    
    def search(self, query, top_k=10):
        """Hybrid search: combine vector and keyword results"""
        # Vector search
        query_embedding = self.embedder.embed(query)
        vector_results = self.vector_store.search(query_embedding, top_k=top_k * 2)
        
        # Keyword search (BM25)
        keyword_results = self.keyword_index.search(query, top_k=top_k * 2)
        
        # Normalize scores
        vector_results = self.normalize_scores(vector_results)
        keyword_results = self.normalize_scores(keyword_results)
        
        # Combine scores
        combined_scores = {}
        for result in vector_results:
            doc_id = result['id']
            combined_scores[doc_id] = {'doc': result, 'score': self.alpha * result['score']}
        
        for result in keyword_results:
            doc_id = result['id']
            keyword_score = (1 - self.alpha) * result['score']
            combined_scores[doc_id] = combined_scores.get(doc_id, {'doc': result, 'score': 0})
            combined_scores[doc_id]['score'] += keyword_score
        
        # Sort by combined score
        ranked_results = sorted(combined_scores.values(), key=lambda x: x['score'], reverse=True)
        return [item['doc'] for item in ranked_results[:top_k]]
```

### Metadata Filtering

```python
class MetadataFilter:
    """Filter retrieval results by metadata"""
    def __init__(self):
        self.filters = {}
    
    def add_filter(self, field, operator, value):
        """Add filter condition"""
        self.filters[field] = {'operator': operator, 'value': value}
    
    def apply_filters(self, results):
        """Apply filters to results"""
        filtered = results
        for field, condition in self.filters.items():
            operator = condition['operator']
            value = condition['value']
            
            if operator == 'eq':
                filtered = [r for r in filtered if r['metadata'].get(field) == value]
            elif operator == 'gt':
                filtered = [r for r in filtered if r['metadata'].get(field, 0) > value]
            elif operator == 'in':
                filtered = [r for r in filtered if r['metadata'].get(field) in value]
        
        return filtered
```

### Query Understanding & Expansion

```python
class QueryUnderstanding:
    """Understand and optimize queries for better retrieval"""
    def __init__(self, llm):
        self.llm = llm
    
    def analyze_query(self, query):
        """Analyze query to understand intent and entities"""
        return {
            'intent': self.classify_intent(query),
            'entities': self.extract_entities(query),
            'keywords': self.extract_keywords(query)
        }
    
    def classify_intent(self, query):
        """Classify user intent"""
        prompt = f"Classify the intent of this query: {query}\n\nPossible intents: factual, procedural, comparative, troubleshooting, general\n\nReturn only the intent."
        return self.llm.generate(prompt).strip()
    
    def extract_entities(self, query):
        """Extract key entities"""
        prompt = f"Extract key entities from: {query}\n\nReturn as JSON: {{'entities': ['entity1', 'entity2']}}"
        response = self.llm.generate(prompt)
        return json.loads(response)['entities']
    
    def expand_query(self, query):
        """Generate query variations for better retrieval"""
        prompt = f"Generate 3 different ways to ask: {query}\n\nReturn as JSON: {{'queries': ['q1', 'q2', 'q3']}}"
        response = self.llm.generate(prompt)
        return json.loads(response)['queries']
```

### Retrieval Evaluation

```python
class RetrievalEvaluator:
    """Evaluate retrieval quality"""
    def evaluate_retrieval(self, query, retrieved_docs, relevant_docs):
        """Evaluate retrieval performance"""
        retrieved_ids = {doc['id'] for doc in retrieved_docs}
        relevant_ids = {doc['id'] for doc in relevant_docs}
        
        true_positives = len(retrieved_ids & relevant_ids)
        precision = true_positives / len(retrieved_docs) if retrieved_docs else 0
        recall = true_positives / len(relevant_docs) if relevant_docs else 0
        f1 = 2 * (precision * recall) / (precision + recall) if (precision + recall) > 0 else 0
        
        return {
            'precision': precision,
            'recall': recall,
            'f1_score': f1,
            'mrr': self.calculate_mrr(retrieved_docs, relevant_docs),
            'ndcg': self.calculate_ndcg(retrieved_docs, relevant_docs)
        }
```

---

## Handling Data Freshness & Changes

### Data Freshness Strategies

#### Strategy 1: Incremental Updates

```python
class IncrementalUpdateManager:
    """Manage incremental updates to knowledge base"""
    def __init__(self, vector_store, document_store):
        self.vector_store = vector_store
        self.document_store = document_store
        self.update_log = []
    
    def update_document(self, document_id, new_content):
        """Update single document"""
        old_doc = self.document_store.get(document_id)
        
        if old_doc['content'] == new_content:
            return {'status': 'NO_CHANGE', 'document_id': document_id}
        
        # Delete old chunks
        old_chunks = self.vector_store.get_chunks(document_id)
        for chunk in old_chunks:
            self.vector_store.delete(chunk['id'])
        
        # Chunk and embed new content
        chunks = self.chunker.chunk(new_content)
        embeddings = self.embedder.embed_batch([c.text for c in chunks])
        
        # Store new chunks
        for i, (chunk, embedding) in enumerate(zip(chunks, embeddings)):
            self.vector_store.add(
                id=f"{document_id}_chunk_{i}",
                vector=embedding,
                metadata={
                    'document_id': document_id,
                    'chunk_index': i,
                    'text': chunk.text,
                    'updated_at': datetime.now().isoformat()
                }
            )
        
        self.document_store.update(document_id, {
            'content': new_content,
            'updated_at': datetime.now().isoformat()
        })
        
        return {'status': 'UPDATED', 'document_id': document_id, 'chunks_updated': len(chunks)}
```

#### Strategy 2: Scheduled Reindexing

```python
class ScheduledReindexer:
    """Periodically reindex knowledge base"""
    def __init__(self, config):
        self.config = config
        self.schedule = config.get('schedule', '0 2 * * *')  # Daily at 2 AM
        self.last_reindex = None
    
    def should_reindex(self):
        """Check if reindexing is needed"""
        if not self.last_reindex:
            return True
        
        next_reindex = self.get_next_scheduled_time()
        if datetime.now() >= next_reindex:
            return True
        
        updates_since_reindex = self.count_updates_since(self.last_reindex)
        if updates_since_reindex >= self.config.get('max_updates_before_reindex', 1000):
            return True
        
        return False
    
    def reindex(self):
        """Perform full reindex"""
        print("Starting full reindex...")
        start_time = datetime.now()
        
        # Backup current index
        self.backup_current_index()
        
        # Get all documents
        documents = self.document_store.get_all()
        
        # Clear and re-ingest
        self.vector_store.clear()
        total_chunks = sum(self.ingest_document(doc) for doc in documents)
        
        self.last_reindex = datetime.now()
        duration = (datetime.now() - start_time).total_seconds()
        
        return {
            'status': 'COMPLETED',
            'documents': len(documents),
            'chunks': total_chunks,
            'duration': duration
        }
```

#### Strategy 3: Change Data Capture (CDC)

```python
class ChangeDataCapture:
    """Capture and propagate changes from source systems"""
    def __init__(self, source_db, vector_store):
        self.source_db = source_db
        self.vector_store = vector_store
        self.cdc_stream = None
    
    def setup_cdc(self):
        """Setup change data capture"""
        self.cdc_stream = self.source_db.get_cdc_stream(
            tables=['documents', 'policies'],
            operations=['INSERT', 'UPDATE', 'DELETE']
        )
        self.cdc_stream.subscribe(self.handle_change)
    
    def handle_change(self, change_event):
        """Handle database change event"""
        operation = change_event['operation']
        data = change_event['data']
        
        if operation == 'INSERT':
            self.handle_insert(data)
        elif operation == 'UPDATE':
            self.handle_update(data)
        elif operation == 'DELETE':
            self.handle_delete(data)
```

### Data Drift Detection

```python
class DataDriftDetector:
    """Detect drift in knowledge base"""
    def __init__(self, vector_store, embedding_model):
        self.vector_store = vector_store
        self.embedding_model = embedding_model
        self.baseline_distribution = None
        self.drift_threshold = 0.15
    
    def establish_baseline(self, sample_size=1000):
        """Establish baseline distribution"""
        sample_docs = self.vector_store.random_sample(sample_size)
        embeddings = [doc['embedding'] for doc in sample_docs]
        
        self.baseline_distribution = {
            'mean': np.mean(embeddings, axis=0),
            'std': np.std(embeddings, axis=0),
            'sample_size': len(embeddings)
        }
    
    def detect_drift(self, current_sample_size=100):
        """Detect drift from baseline"""
        if not self.baseline_distribution:
            raise ValueError("Baseline not established")
        
        current_docs = self.vector_store.random_sample(current_sample_size)
        current_embeddings = [doc['embedding'] for doc in current_docs]
        current_mean = np.mean(current_embeddings, axis=0)
        
        # Calculate PSI
        psi = self.calculate_psi(self.baseline_distribution['mean'], current_mean)
        
        drift_detected = psi > self.drift_threshold
        
        return {
            'drift_detected': drift_detected,
            'psi_score': psi,
            'recommendation': self.get_recommendation(psi)
        }
    
    def calculate_psi(self, baseline, current, buckets=10):
        """Calculate Population Stability Index"""
        breaks = np.linspace(min(baseline.min(), current.min()),
                           max(baseline.max(), current.max()),
                           buckets + 1)
        
        baseline_hist, _ = np.histogram(baseline, bins=breaks)
        current_hist, _ = np.histogram(current, bins=breaks)
        
        epsilon = 1e-10
        baseline_pct = np.maximum(baseline_hist / len(baseline), epsilon)
        current_pct = np.maximum(current_hist / len(current), epsilon)
        
        psi = np.sum((current_pct - baseline_pct) * np.log(current_pct / baseline_pct))
        return psi
    
    def get_recommendation(self, psi_score):
        """Get recommendation based on drift score"""
        if psi_score < 0.1:
            return "No action needed - minimal drift"
        elif psi_score < 0.2:
            return "Monitor - slight drift detected"
        elif psi_score < 0.3:
            return "Investigate - moderate drift, consider reindexing"
        else:
            return "Action required - significant drift, reindex recommended"
```

### Versioning and Rollback

```python
class KnowledgeBaseVersioning:
    """Version control for knowledge base"""
    def __init__(self, vector_store, object_storage):
        self.vector_store = vector_store
        self.object_storage = object_storage
        self.versions = {}
    
    def create_snapshot(self, version_name):
        """Create snapshot of current knowledge base"""
        snapshot = {
            'version': version_name,
            'timestamp': datetime.now().isoformat(),
            'vector_store': self.vector_store.export(),
            'document_store': self.document_store.export()
        }
        
        self.object_storage.put(f"snapshots/{version_name}.json", json.dumps(snapshot))
        self.versions[version_name] = {'timestamp': snapshot['timestamp']}
        
        return {'version': version_name, 'status': 'CREATED'}
    
    def rollback(self, version_name):
        """Rollback to previous version"""
        if version_name not in self.versions:
            raise ValueError(f"Version {version_name} not found")
        
        snapshot_data = self.object_storage.get(f"snapshots/{version_name}.json")
        snapshot = json.loads(snapshot_data)
        
        # Backup current state
        current_version = f"pre_rollback_{datetime.now().strftime('%Y%m%d_%H%M%S')}"
        self.create_snapshot(current_version)
        
        # Restore
        self.vector_store.clear()
        self.vector_store.import_data(snapshot['vector_store'])
        
        return {'status': 'ROLLED_BACK', 'version': version_name}
```

---

## Hands-On Exercises

### Exercise 1: Build a Simple RAG System

**Objective:** Build a basic RAG system for document Q&A

**Task:**
1. Create a document ingestion pipeline
2. Implement vector storage using ChromaDB
3. Build a retrieval system
4. Create a query interface

**Solution:**

```python
# Step 1: Setup
import chromadb
from sentence_transformers import SentenceTransformer

client = chromadb.Client()
collection = client.create_collection("documents")
embedder = SentenceTransformer('all-MiniLM-L6-v2')

# Step 2: Document Ingestion
def ingest_documents(documents):
    for doc in documents:
        chunks = chunk_document(doc['text'], chunk_size=200, overlap=50)
        embeddings = embedder.encode([c['text'] for c in chunks])
        
        for i, (chunk, embedding) in enumerate(zip(chunks, embeddings)):
            collection.add(
                documents=[chunk['text']],
                embeddings=[embedding.tolist()],
                metadatas=[{'source': doc['source']}],
                ids=[f"{doc['id']}_chunk_{i}"]
            )

# Step 3: Query
def query(question, top_k=3):
    question_embedding = embedder.encode(question)
    results = collection.query(query_embeddings=[question_embedding.tolist()], n_results=top_k)
    
    context = "\n\n".join(results['documents'][0])
    
    return {
        'answer': 'LLM would generate answer here',
        'sources': [m['source'] for m in results['metadatas'][0]],
        'context': context
    }

# Usage
documents = [
    {'id': '1', 'source': 'policy.pdf', 'text': 'Company refund policy...'},
    {'id': '2', 'source': 'handbook.pdf', 'text': 'Employee handbook...'}
]

ingest_documents(documents)
result = query("What is the refund policy?")
```

### Exercise 2: Implement Chunking Strategies

**Objective:** Compare different chunking strategies

**Task:**
1. Implement fixed-size, semantic, and recursive chunking
2. Test on sample documents
3. Compare retrieval quality

**Solution:**

```python
class FixedSizeChunker:
    def chunk(self, text, chunk_size=200, overlap=50):
        tokens = text.split()
        chunks = []
        start = 0
        while start < len(tokens):
            end = start + chunk_size
            chunk_text = ' '.join(tokens[start:end])
            chunks.append(chunk_text)
            start = end - overlap
        return chunks

class SemanticChunker:
    def chunk(self, text, max_chunk_size=200):
        paragraphs = text.split('\n\n')
        chunks = []
        current_chunk = []
        current_size = 0
        
        for para in paragraphs:
            para_size = len(para.split())
            if current_size + para_size > max_chunk_size and current_chunk:
                chunks.append(' '.join(current_chunk))
                current_chunk = []
                current_size = 0
            current_chunk.append(para)
            current_size += para_size
        
        if current_chunk:
            chunks.append(' '.join(current_chunk))
        
        return chunks

# Test
sample_doc = """
Machine learning is a subset of artificial intelligence. 
It focuses on building systems that learn from data.

Deep learning is a type of machine learning that uses neural networks 
with many layers. These networks can learn complex patterns.
"""

fixed_chunker = FixedSizeChunker()
semantic_chunker = SemanticChunker()

fixed_chunks = fixed_chunker.chunk(sample_doc, chunk_size=20, overlap=5)
semantic_chunks = semantic_chunker.chunk(sample_doc, max_chunk_size=30)
```

### Exercise 3: Design a Hybrid RAG System

**Objective:** Combine vector and graph RAG

**Solution:**

```python
class HybridRAGSystem:
    def __init__(self, vector_store, knowledge_graph, llm):
        self.vector_store = vector_store
        self.kg = knowledge_graph
        self.llm = llm
    
    def query(self, question):
        # Vector search
        query_embedding = self.embedder.embed(question)
        vector_results = self.vector_store.search(query_embedding, top_k=10)
        
        # Graph search
        entities = self.extract_entities(question)
        graph_results = []
        for entity in entities:
            subgraph = self.kg.get_subgraph(entity, max_hops=2)
            graph_results.append(self.subgraph_to_text(subgraph))
        
        # Combine using RRF
        combined = self.reciprocal_rank_fusion(vector_results, graph_results)
        
        # Generate answer
        context = "\n\n".join(combined)
        prompt = f"Context: {context}\n\nQuestion: {question}\n\nAnswer:"
        answer = self.llm.generate(prompt)
        
        return {'answer': answer, 'vector_sources': [r['source'] for r in vector_results[:3]]}
```

### Exercise 4: Implement Context Engineering

**Objective:** Optimize context for LLM

**Solution:**

```python
class ContextOptimizer:
    def __init__(self, max_tokens=4000):
        self.max_tokens = max_tokens
        self.cache = {}
    
    def optimize_context(self, query, documents, user_prefs):
        cache_key = hash(query + str(user_prefs))
        if cache_key in self.cache:
            return self.cache[cache_key]
        
        selected_docs = documents[:3]
        context_parts = []
        total_tokens = 0
        
        for doc in selected_docs:
            doc_tokens = len(doc['text'].split())
            if total_tokens + doc_tokens > self.max_tokens:
                compressed = self.compress_text(doc['text'], self.max_tokens - total_tokens)
                context_parts.append(compressed)
                break
            context_parts.append(doc['text'])
            total_tokens += doc_tokens
        
        context = "\n\n".join(context_parts)
        self.cache[cache_key] = context
        
        return context
```

### Exercise 5: Build a Complete RAG Pipeline

**Objective:** Integrate all components into production-ready system

**Solution:**

See complete implementation in the RAG System section above, plus monitoring and evaluation:

```python
class RAGMonitor:
    def log_query(self, query, results, latency):
        # Log to monitoring system
        pass

class RAGEvaluator:
    def evaluate(self, test_cases):
        metrics = []
        for test in test_cases:
            result = self.rag_system.query(test['question'])
            metrics.append(self.calculate_metrics(result, test['expected']))
        return metrics
```

### Exercise 6: RAG Failure Analysis

**Objective:** Identify and fix common RAG failures

**Solution:**

```python
failure_analysis = {
    'scenario_a': {
        'root_cause': 'Poor prompt engineering or context assembly',
        'fixes': ['Improve system prompt', 'Add explicit citations', 'Use better LLM']
    },
    'scenario_b': {
        'root_cause': 'Poor retrieval quality',
        'fixes': ['Improve chunking', 'Use better embeddings', 'Add reranking', 'Implement hybrid search']
    },
    'scenario_c': {
        'root_cause': 'Single-hop retrieval insufficient',
        'fixes': ['Implement multi-hop retrieval', 'Add query decomposition', 'Use Graph RAG']
    },
    'scenario_d': {
        'root_cause': 'Performance bottleneck',
        'fixes': ['Add caching', 'Use faster embedding model', 'Optimize vector store', 'Reduce context size']
    }
}
```

---

## Practice Question Bank

### Multiple Choice Questions

**1. What is the primary benefit of RAG over fine-tuning?**
A) Lower cost  
B) Real-time knowledge updates without retraining  
C) Better accuracy  
D) Faster inference  

**Answer: B**  
**Explanation:** RAG allows updating knowledge by modifying the knowledge base without retraining the model.

---

**2. Which chunking strategy preserves semantic boundaries best?**
A) Fixed-size chunking  
B) Semantic chunking  
C) Character-based chunking  
D) Random chunking  

**Answer: B**  
**Explanation:** Semantic chunking splits at natural boundaries like paragraphs and sections.

---

**3. What is the purpose of reranking in RAG?**
A) Speed up retrieval  
B) Improve retrieval relevance by reordering results  
C) Reduce costs  
D) Compress context  

**Answer: B**  
**Explanation:** Reranking uses a more sophisticated model to reorder results, improving relevance.

---

**4. When should you use Graph RAG over Vector RAG?**
A) Always  
B) For simple factual queries  
C) For complex multi-hop reasoning  
D) Never  

**Answer: C**  
**Explanation:** Graph RAG excels at multi-hop reasoning and complex relationship queries.

---

**5. What is the main challenge with context windows in RAG?**
A) They're too large  
B) Limited token budget requires careful context selection  
C) They don't support images  
D) They're too slow  

**Answer: B**  
**Explanation:** LLMs have limited context windows, requiring careful selection of what to include.

---

**6. Which is NOT a type of memory in AI systems?**
A) Ephemeral memory  
B) Short-term memory  
C) Long-term memory  
D) Permanent memory  

**Answer: D**  
**Explanation:** The three main types are ephemeral, short-term, and long-term. "Permanent memory" is not standard.

---

**7. What is the purpose of hybrid search in RAG?**
A) Combine vector and keyword search for better results  
B) Search multiple databases  
C) Use multiple LLMs  
D) Combine RAG with fine-tuning  

**Answer: A**  
**Explanation:** Hybrid search combines vector (semantic) search with keyword (BM25) search.

---

**8. What is data drift in RAG systems?**
A) Data moving between servers  
B) Changes in data distribution over time  
C) Data being deleted  
D) Data being corrupted  

**Answer: B**  
**Explanation:** Data drift occurs when the distribution of data changes over time.

---

**9. What is the recommended approach for handling data freshness?**
A) Reindex everything daily  
B) Use incremental updates with periodic full reindexing  
C) Never update the knowledge base  
D) Manual updates only  

**Answer: B**  
**Explanation:** Incremental updates handle changes efficiently, while periodic reindexing ensures consistency.

---

**10. What is Reciprocal Rank Fusion (RRF)?**
A) A method to combine results from multiple retrievers  
B) A ranking algorithm for single retriever  
C) A compression technique  
D) A caching strategy  

**Answer: A**  
**Explanation:** RRF combines ranked results from multiple retrieval methods.

---

### Scenario-Based Questions

**11. Scenario:** Your RAG system retrieves documents with 90% precision but users complain answers are often wrong. What's the likely issue?

A) Retrieval quality is poor  
B) LLM is generating incorrect answers from correct context  
C) Need more documents  
D) Need better embeddings  

**Answer: B**  
**Explanation:** High precision means retrieval is working well. The issue is in generation.

---

**12. Scenario:** You need to build a RAG system for legal document search with complex relationships. Which approach?

A) Simple vector RAG  
B) Graph RAG  
C) No RAG needed  
D) Fine-tuning only  

**Answer: B**  
**Explanation:** Legal documents have complex relationships that Graph RAG handles better.

---

**13. Scenario:** Your RAG system latency is 8 seconds. What's the best optimization approach?

A) Use a faster LLM  
B) Add caching, optimize retrieval, use smaller context  
C) Reduce document count  
D) Switch to fine-tuning  

**Answer: B**  
**Explanation:** Multi-faceted approach: cache, optimize retrieval, reduce context size.

---

**14. Scenario:** Knowledge base has 100K documents, updates hourly. What's the best freshness strategy?

A) Full reindex every hour  
B) Incremental updates with CDC  
C) Manual updates  
D) Weekly reindex  

**Answer: B**  
**Explanation:** CDC with incremental updates is most efficient for large, frequently updated knowledge bases.

---

**15. Scenario:** Users ask follow-up questions that reference earlier conversation. What memory type should you use?

A) Ephemeral memory  
B) Long-term memory  
C) Session memory  
D) No memory needed  

**Answer: C**  
**Explanation:** Session memory persists across multiple interactions within a user session.

---

### True/False Questions

**16. RAG completely eliminates hallucinations.**  
**Answer: False**  
**Explanation:** RAG reduces hallucinations but doesn't eliminate them entirely.

---

**17. Graph RAG is always better than Vector RAG.**  
**Answer: False**  
**Explanation:** Graph RAG is better for complex relationships but more complex and slower.

---

**18. Chunking doesn't affect RAG quality.**  
**Answer: False**  
**Explanation:** Chunking significantly affects retrieval quality.

---

**19. Data drift is not a concern for RAG systems.**  
**Answer: False**  
**Explanation:** Data drift is a major concern that degrades retrieval quality.

---

**20. Reranking always improves retrieval quality.**  
**Answer: False**  
**Explanation:** Reranking adds latency and may not help for simple queries.

---

### Short Answer Questions

**21. Explain the trade-offs between fixed-size and semantic chunking.**

**Answer:**

**Fixed-size:** Simple, predictable, fast, but can break semantic units. Best for homogeneous documents.

**Semantic:** Preserves semantic coherence, but variable sizes and more complex. Best for quality-critical applications.

---

**22. How would you design a RAG system for a 1M document knowledge base that updates daily?**

**Answer:**

1. Incremental updates using CDC
2. Distributed vector store (Pinecone/Weaviate)
3. Recursive chunking (512 tokens, 50 overlap)
4. Hybrid search (vector + keyword)
5. Cross-encoder reranking
6. Redis caching
7. Full reindex weekly

---

**23. What is context engineering and why does it matter?**

**Answer:**

Context engineering is designing and optimizing information for LLMs. It matters because:
1. Limited context windows require careful selection
2. Quality over quantity
3. Cost implications
4. Performance impact
5. Accuracy depends on good context

---

**24. Compare Vector RAG and Graph RAG for enterprise knowledge management.**

**Answer:**

**Vector RAG:** Fast, simple, scalable. Best for document search, FAQ systems.

**Graph RAG:** Relationship reasoning, multi-hop queries, explainable. Best for organizational knowledge, complex dependencies.

**Recommendation:** Start with Vector RAG, add Graph RAG for specific use cases.

---

**25. Design a monitoring strategy for a production RAG system.**

**Answer:**

**Metrics:**
- System: Latency, availability, error rate, throughput
- Retrieval: Precision/recall, no results rate, relevance score
- Response: CSAT, hallucination rate, citation accuracy
- Knowledge Base: Document count, age, drift score
- Cost: API costs, infrastructure, cost per query

**Alerting:** Latency >2s, error rate >1%, no results >10%, drift >0.3

---

## Self-Assessment Checklist

### Core Concepts

- [ ] I can explain what RAG is and why it's important
- [ ] I understand the RAG architecture and components
- [ ] I can compare vector RAG vs. graph RAG
- [ ] I know when to use each RAG approach
- [ ] I understand context engineering principles
- [ ] I can design memory systems for AI applications
- [ ] I know how to deploy knowledge graphs
- [ ] I can optimize retrieval quality
- [ ] I understand data freshness challenges
- [ ] I can implement drift detection

### Practical Skills

- [ ] I can build a basic RAG system
- [ ] I can implement different chunking strategies
- [ ] I can select appropriate embedding models
- [ ] I can implement hybrid search
- [ ] I can add reranking to improve quality
- [ ] I can design context assembly strategies
- [ ] I can implement memory systems
- [ ] I can build a knowledge graph
- [ ] I can detect data drift
- [ ] I can implement incremental updates

### Knowledge Check

Score yourself (5 = expert, 3 = proficient, 1 = beginner):

1. RAG fundamentals: ___/5
2. Chunking strategies: ___/5
3. Vector vs Graph RAG: ___/5
4. Context engineering: ___/5
5. Memory systems: ___/5
6. Knowledge graphs: ___/5
7. Data freshness: ___/5
8. Production considerations: ___/5

**Overall Score:** ___/40

**Interpretation:**
- 32-40: Ready to move to Week 3
- 24-31: Review weak areas before proceeding
- <24: Re-study Week 2 materials

---

## Summary & Key Takeaways

### Week 2 in 60 Seconds

**RAG** is the workhorse pattern of enterprise AI, grounding LLM responses in external knowledge bases.

**Key Principles:**
1. **RAG Architecture:** Ingest → Chunk → Embed → Store → Retrieve → Assemble → Generate
2. **Chunking Matters:** Choose strategy based on document type
3. **Hybrid Search:** Combine vector + keyword for best results
4. **Context Engineering:** Optimize what goes into context window
5. **Memory Systems:** Ephemeral, session, and long-term memory
6. **Graph RAG:** Use for complex relationships
7. **Data Freshness:** Implement incremental updates and drift detection

**Critical Insights:**
✅ RAG reduces hallucinations by grounding in retrieved facts
✅ Chunking strategy significantly impacts retrieval quality
✅ Hybrid search combines benefits of vector and keyword search
✅ Context engineering maximizes limited context windows
✅ Data drift degrades performance - monitor and reindex

### Looking Ahead to Week 3

Next week: **AI Agents** - building systems that can take actions, use tools, and operate autonomously.

**Homework:** Build a retrieval architecture for a complex query scenario. Justify design choices and identify the biggest anticipated failure mode.

---

## Further Reading

### Essential Reading

1. **"Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"** - Lewis et al. (2020)
   - Link: https://arxiv.org/abs/2005.11401
   - Why: Original RAG paper

2. **"LangChain Documentation"**
   - Link: https://python.langchain.com/
   - Why: Comprehensive RAG implementation guide

3. **"LlamaIndex Documentation"**
   - Link: https://docs.llamaindex.ai/
   - Why: Data framework for LLMs and RAG

### Tools & Frameworks

**LLM Frameworks:** LangChain, LlamaIndex, Semantic Kernel  
**Vector Databases:** Pinecone, Weaviate, ChromaDB, Qdrant  
**Embedding Models:** OpenAI, Cohere, Sentence Transformers, Voyage AI  
**Graph Databases:** Neo4j, Amazon Neptune, JanusGraph  
**Monitoring:** LangSmith, Arize, Weights & Biases

### Communities

- r/LangChain - LangChain discussions
- r/MachineLearning - ML research
- AI Engineering Discord - Real-time support

---

**🎯 Week 2 Complete! You now understand RAG and context pipelines - the foundation of enterprise AI.**

**➡️ Next:** [Week 3 - Designing & Building AI Agents](Week-03-AI-Agents-Complete-Guide.md)

---

*Estimated Reading Time:* 4-5 hours  
*Exercises Completion Time:* 4-5 hours  
*Total Time:* 10-12 hours