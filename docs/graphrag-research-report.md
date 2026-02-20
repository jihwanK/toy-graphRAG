# GraphRAG Toy Project: Complete Research Report

> Generated 2026-02-20 by a 4-agent research team

---

## Part 1: What is GraphRAG?

### Traditional RAG vs GraphRAG

| Aspect | Traditional RAG | GraphRAG |
|--------|----------------|----------|
| **Knowledge storage** | Flat text chunks → vector embeddings | Entity-relationship graph + community summaries |
| **Retrieval** | Semantic similarity (cosine distance) | Graph traversal + community summaries + vector hybrid |
| **Strength** | Simple factual lookups | Multi-hop reasoning, connecting dots across sources |
| **Weakness** | Struggles with "connect the dots" queries | Higher setup cost, more LLM calls during indexing |
| **Analogy** | Library card catalog (find by keyword) | Mind map (follow relationships between ideas) |

### Core Architecture (Microsoft's Approach)

The pipeline from the [foundational paper](https://arxiv.org/abs/2404.16130):

```
Raw Text Documents
       ↓
[1] Entity & Relationship Extraction (LLM)
       ↓
[2] Knowledge Graph Construction (nodes = entities, edges = relationships)
       ↓
[3] Community Detection (Leiden algorithm)
       ↓
[4] Community Summarization (LLM generates summaries per community)
       ↓
[5] Query → Local Search (entity-focused) OR Global Search (theme-focused)
       ↓
Answer with structured context
```

**Key concepts mapped to Data Science background:**
- **Entity extraction** → Like NER in spaCy, but LLM-powered for richer relationship capture
- **Community detection (Leiden)** → Similar to graph clustering (spectral clustering, but on knowledge graphs). Finds groups of tightly-connected entities
- **Local search** → Retrieves specific entity neighborhoods (good for "tell me about X" queries)
- **Global search** → Uses community summaries via map-reduce over the entire graph (good for "what are the main themes?" queries)

### When GraphRAG Shines vs Overkill

**Use GraphRAG when:**
- Questions require connecting information across multiple documents
- Your domain has rich entity relationships (people, organizations, events, concepts)
- You need to answer "big picture" summarization questions
- Multi-hop reasoning is needed ("Who worked with X, who later joined Y?")

**Stick with traditional RAG when:**
- Simple factual Q&A ("What is the capital of France?")
- Your data has no meaningful entity relationships
- You need low latency and minimal setup cost
- Your corpus is small and homogeneous

---

## Part 2: Toy Project Ideas (Ranked)

### 1. Scientific Paper Explorer (Recommended)
- **Description**: Build a knowledge graph from ML/AI paper abstracts + metadata. Query relationships between authors, methods, datasets, and findings.
- **Why GraphRAG**: Papers have rich entity relationships (authors → institutions, methods → datasets, papers → citations). Multi-hop: "What methods did researchers from Stanford use on ImageNet?"
- **Complexity**: Intermediate
- **Key concepts**: Entity extraction, community detection, local + global search
- **Dataset**: ArXiv abstracts

### 2. Movie/TV Show Recommendation Graph
- **Description**: Knowledge graph of movies, actors, directors, genres, reviews. Ask complex questions like "Find sci-fi movies by directors who also made critically acclaimed dramas."
- **Why GraphRAG**: Natural entity-relationship structure. Multi-hop reasoning across cast, genre, ratings.
- **Complexity**: Beginner (great first project)
- **Dataset**: Neo4j's built-in Movies dataset, TMDB dataset

### 3. News Event Tracker
- **Description**: Knowledge graph from news articles. Track entities (people, organizations, locations) and how they relate across events over time.
- **Why GraphRAG**: News is inherently relational. Global search excels at "What are the main themes this week?"
- **Complexity**: Intermediate
- **Dataset**: BBC News dataset, or scrape RSS feeds

### 4. Recipe & Ingredient Knowledge Graph
- **Description**: Graph connecting recipes, ingredients, cuisines, dietary restrictions, nutritional info. Query: "What vegetarian Italian dishes use ingredients common in Thai food?"
- **Why GraphRAG**: Multi-hop reasoning across ingredients, cuisines, dietary categories
- **Complexity**: Beginner-Intermediate
- **Dataset**: Recipe1M, Food.com Kaggle dataset

### 5. Code Documentation Navigator
- **Description**: Index a codebase's documentation and build a knowledge graph of modules, functions, dependencies. Ask "How does module X relate to module Y?"
- **Why GraphRAG**: Code has natural graph structure (imports, calls, inheritance)
- **Complexity**: Advanced
- **Dataset**: Any open-source project's docs

### 6. Healthcare Concept Explorer
- **Description**: Knowledge graph of diseases, symptoms, treatments, and drugs from medical literature.
- **Why GraphRAG**: Medical knowledge is highly relational. "What treatments for condition X also address symptom Y?"
- **Complexity**: Advanced
- **Dataset**: PubMed abstracts, SNOMED CT subset

---

## Part 3: Datasets

### Pre-processed (Graph-Ready)

| Dataset | Format | Size | Source | Best For |
|---------|--------|------|--------|----------|
| **Neo4j Movies** | Neo4j-native | ~170 nodes | [Built-in `:PLAY movies`](https://neo4j.com/docs/getting-started/appendix/example-data/) | Learning Cypher, first steps |
| **FB15k-237** | Triplets (TSV) | 310K triplets, 14.5K entities | [HuggingFace](https://huggingface.co/datasets/KGraph/FB15k-237) | Knowledge graph embedding, link prediction |
| **WN18RR** | Triplets (TSV) | 93K triplets, 41K entities | [GitHub](https://github.com/villmow/datasets_knowledge_embedding) | Smaller graph experiments |
| **ConceptNet** (subset) | Triplets (CSV) | 34M relationships (use subset) | [HuggingFace](https://huggingface.co/datasets/conceptnet5) | Common-sense reasoning graph |
| **Neo4j Sandbox datasets** | Neo4j-native | Various | [Neo4j Sandbox](https://neo4j.com/docs/getting-started/appendix/example-data/) | Free 3-day cloud instances |

### Semi-structured (Easy to Convert)

| Dataset | Format | Size | Source | Processing Needed |
|---------|--------|------|--------|--------------------|
| **TMDB Movies** | JSON/CSV | ~50K movies | Kaggle | Extract entities: Movie→Actor, Movie→Director, Movie→Genre |
| **ArXiv Metadata** | JSON | ~2M papers | [Kaggle](https://www.kaggle.com/datasets/Cornell-University/arxiv) | Extract: Paper→Author, Paper→Category, paper citations |
| **Food.com Recipes** | CSV | ~230K recipes | Kaggle | Extract: Recipe→Ingredient, Recipe→Cuisine, Ingredient→Nutrition |

### Raw (Used in Microsoft GraphRAG Examples)

| Dataset | Notes |
|---------|-------|
| **A Christmas Carol** (Dickens) | Default example in Microsoft's GraphRAG tutorial |
| **Wikipedia articles** | Download specific topic pages for focused experiments |
| **Your own PDFs/docs** | GraphRAG shines on domain-specific documents |

---

## Part 4: Tools & Practical Setup

### Recommended Tech Stack

```
Graph Database:     Neo4j (via Docker or AuraDB Free)
GraphRAG Framework: Microsoft GraphRAG (graphrag Python package)
Alternative:        LlamaIndex PropertyGraphIndex + Neo4j
LLM:                OpenAI GPT-4o-mini (cheap) or Ollama (free/local)
Embeddings:         text-embedding-3-small (OpenAI) or nomic-embed (local)
Visualization:      Neo4j Browser + pyvis for Python
Language:           Python 3.11+
```

### Neo4j Setup

```bash
# Option A: Docker (recommended for local dev)
docker run -d --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/your-password \
  neo4j:latest

# Option B: AuraDB Free (cloud, zero setup)
# → https://neo4j.com/cloud/aura-free/
# Free tier: 50K nodes, 175K relationships
```

### Cypher Basics

```cypher
-- Create nodes
CREATE (m:Movie {title: "The Matrix", year: 1999})
CREATE (p:Person {name: "Keanu Reeves"})

-- Create relationship
MATCH (p:Person {name: "Keanu Reeves"}), (m:Movie {title: "The Matrix"})
CREATE (p)-[:ACTED_IN {role: "Neo"}]->(m)

-- Multi-hop query
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
WHERE p.name = "Keanu Reeves"
RETURN d.name AS director, m.title AS movie
```

### Python + Neo4j

```python
from neo4j import GraphDatabase

driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "your-password"))

with driver.session() as session:
    result = session.run(
        "MATCH (p:Person)-[:ACTED_IN]->(m:Movie) RETURN p.name, m.title LIMIT 10"
    )
    for record in result:
        print(record["p.name"], "→", record["m.title"])
```

### Microsoft GraphRAG Setup

```bash
pip install graphrag

mkdir my_graphrag_project && cd my_graphrag_project
graphrag init  # Creates settings.yaml, .env, input/ folder

# Put text files in input/ folder, then:
graphrag index                                          # Build knowledge graph
graphrag query --method global "What are the main themes?"
graphrag query --method local "Tell me about character X"
```

### LlamaIndex + Neo4j Alternative

```python
from llama_index.core import SimpleDirectoryReader, PropertyGraphIndex
from llama_index.graph_stores.neo4j import Neo4jPropertyGraphStore

graph_store = Neo4jPropertyGraphStore(
    username="neo4j", password="your-password",
    url="bolt://localhost:7687"
)

documents = SimpleDirectoryReader("./data").load_data()
index = PropertyGraphIndex.from_documents(
    documents, property_graph_store=graph_store
)

retriever = index.as_retriever()
response = retriever.retrieve("How are X and Y related?")
```

### LangChain + Neo4j Alternative

```python
from langchain_neo4j import Neo4jGraph, GraphCypherQAChain
from langchain_openai import ChatOpenAI

graph = Neo4jGraph(url="bolt://localhost:7687", username="neo4j", password="password")
llm = ChatOpenAI(model="gpt-4o-mini")

chain = GraphCypherQAChain.from_llm(llm=llm, graph=graph, verbose=True)
result = chain.invoke({"query": "Who directed The Matrix?"})
```

### Cost-Effective Local Setup (Ollama)

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2         # Entity extraction & generation
ollama pull nomic-embed-text  # Embeddings
```

See [graphrag-local-ollama](https://github.com/TheAiSingularity/graphrag-local-ollama) for a ready-made setup.

**Trade-off**: Free but ~10-50x slower for indexing.

---

## Part 5: Learning Roadmap

### Phase 1: Foundations (Week 1)
1. Read the [Microsoft GraphRAG paper](https://arxiv.org/abs/2404.16130) — focus on Sections 1-3
2. Run `:PLAY movies` in Neo4j Browser to learn Cypher basics
3. Complete [Neo4j GraphAcademy's free Knowledge Graph course](https://graphacademy.neo4j.com/knowledge-graph-rag/)
4. Explore the [Awesome-GraphRAG](https://github.com/DEEP-PolyU/Awesome-GraphRAG) curated list

### Phase 2: First Implementation (Week 2)
5. Run Microsoft GraphRAG's quickstart with "A Christmas Carol"
6. Experiment with both `local` and `global` query modes
7. Try [GraphRAG-Local-UI](https://github.com/severian42/GraphRAG-Local-UI) for visualization
8. Read the [Analytics Vidhya complete guide](https://www.analyticsvidhya.com/blog/2024/11/graphrag/)

### Phase 3: Build Your Toy Project (Weeks 3-4)
9. Choose your dataset (recommended: ArXiv metadata)
10. Set up Neo4j + Python environment
11. Build the entity extraction pipeline
12. Construct the knowledge graph
13. Implement local + global search
14. Add a simple chat interface

### Phase 4: Deepen Understanding (Ongoing)
15. Compare Microsoft GraphRAG vs LlamaIndex PropertyGraphIndex
16. Experiment with hybrid vector + graph retrieval
17. Try [LightRAG](https://github.com/HKUDS/LightRAG) as a lightweight alternative
18. Read newer papers from the [Awesome-GraphRAG list](https://github.com/DEEP-PolyU/Awesome-GraphRAG)

---

## Part 6: Tips & Pitfalls

### Common Pitfalls
1. **Starting too big**: Don't index 10,000 documents on day 1. Start with 5-10 short texts
2. **Ignoring costs**: Entity extraction calls the LLM per chunk. A 500-page book could cost $10-50 with GPT-4o. Use `gpt-4o-mini` or local models first
3. **Skipping community detection understanding**: The Leiden algorithm is what makes global search powerful
4. **Not comparing against baseline RAG**: Always benchmark GraphRAG against plain RAG

### Bonus Tools
- **[nano-graphrag](https://github.com/gusye1234/nano-graphrag)** — Minimalist implementation (~800 lines). Great for understanding internals
- **[LightRAG](https://github.com/HKUDS/LightRAG)** — Faster, simpler alternative
- **[Neo4j GraphRAG Python](https://github.com/neo4j/neo4j-graphrag-python)** — Official Neo4j package with built-in KG builder
- **[pyvis](https://pyvis.readthedocs.io/)** — Interactive graph visualization in Jupyter

---

## Sources

- [Microsoft GraphRAG Paper (arxiv 2404.16130)](https://arxiv.org/abs/2404.16130)
- [Microsoft GraphRAG GitHub](https://github.com/microsoft/graphrag)
- [Microsoft GraphRAG Getting Started](https://microsoft.github.io/graphrag/get_started/)
- [Neo4j Example Datasets](https://neo4j.com/docs/getting-started/appendix/example-data/)
- [Neo4j GraphRAG Python](https://github.com/neo4j/neo4j-graphrag-python)
- [Neo4j GraphAcademy](https://graphacademy.neo4j.com/knowledge-graph-rag/)
- [LlamaIndex PropertyGraphIndex + Neo4j](https://developers.llamaindex.ai/python/examples/property_graph/property_graph_neo4j/)
- [Awesome-GraphRAG (DEEP-PolyU)](https://github.com/DEEP-PolyU/Awesome-GraphRAG)
- [Awesome-GraphRAG (graphrag org)](https://github.com/graphrag/awesome-graphrag)
- [FB15k-237 on HuggingFace](https://huggingface.co/datasets/KGraph/FB15k-237)
- [ConceptNet on HuggingFace](https://huggingface.co/datasets/conceptnet5)
- [GraphRAG-Local-UI](https://github.com/severian42/GraphRAG-Local-UI)
- [GraphRAG + Ollama](https://github.com/TheAiSingularity/graphrag-local-ollama)
- [GraphRAG Complete Guide (Analytics Vidhya)](https://www.analyticsvidhya.com/blog/2024/11/graphrag/)
- [RAG vs GraphRAG Comparison](https://memgraph.com/blog/rag-vs-graphrag)
- [nano-graphrag](https://github.com/gusye1234/nano-graphrag)
- [LightRAG](https://github.com/HKUDS/LightRAG)
- [GraphRAG Local Ollama Setup Guide](https://chishengliu.com/posts/graphrag-local-ollama/)
