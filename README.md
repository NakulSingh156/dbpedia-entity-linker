#  Neuro-Symbolic Extraction Pipeline (GSoC 2026 Prototype)

A robust, Dockerized **Neuro-Symbolic Architecture** designed to seamlessly integrate with the DBpedia/GSoC25 infrastructure. This prototype solves the "Hallucination" and "Directionality" bottlenecks in standard LLM extractors by pairing Neural embedding models with strict Symbolic Knowledge Graph validation.

## The Core Innovation

Current LLM extractors confidently invent facts (hallucinations) and struggle with linguistic variance (e.g., active vs. passive voice). 
This pipeline introduces a **Hybrid Architecture**:

1. **Neural Layer (`all-MiniLM-L6-v2`):** Handles semantic extraction and linguistic variance.
2. **Infrastructure Layer (Redis):** Completely offline, high-performance Entity Resolution compatible with DBpedia's existing GSoC25 architecture.
3. **Symbolic Layer (SPARQL):** Validates triples against the DBpedia Ontology, acting as a strict "Hallucination Buster."

## System Architecture

* **Dockerized Environment:** Zero-config reproducibility. Isolates the Python extraction environment and the Redis backend.
* **Offline Entity Resolution:** Uses a seeded Redis cache (`seed_redis.py`) to mock DBpedia Anchor Candidates and Types, eliminating slow and rate-limited external API calls.
* **Deterministic Extraction:** Enforces deterministic Subject/Object assignment based on character-index sorting.

## Key Features & Evidence

### 1. The "Hallucination Buster" (Ontology Guardrails)
If the Neural model extracts a factually incorrect triple (e.g., *Ronaldo -> playsFor -> Chicago Bulls*), the Symbolic layer intercepts and rejects it as contradicting world knowledge, protecting the Knowledge Graph.

<img width="1926" height="506" alt="image" src="https://github.com/user-attachments/assets/4e6ffa31-be68-4c9b-ae32-fef8201da070" />


### 2. Client-Side Graph Reasoning (BFS Traversal)
If strict ontology matching fails due to historical/temporal shifts (e.g., *Ronaldo -> Real Madrid*), the system falls back to a BFS traversal, exploiting `dbo:wikiPageWikiLink` to compute contextual relatedness.

<img width="2336" height="214" alt="image" src="https://github.com/user-attachments/assets/caab6569-9194-445c-bbb5-34c90be439d7" />


### 3. Linguistic Robustness
The extraction logic handles syntactic shifts naturally:
* **Active:** "Cristiano Ronaldo plays for Real Madrid." -> `dbo:team`
* **Passive:** "Cristiano Ronaldo was signed by Real Madrid." -> `dbo:team`

##  How to Run (Zero-Config Docker)


```
1. Clone the repository

git clone [https://github.com/NakulSingh156/dbpedia-entity-linker.git](https://github.com/NakulSingh156/dbpedia-entity-linker.git)
cd dbpedia-entity-linker

2. Build the Docker Environment

docker compose up -d --build

3. Seed the Redis Database (Mock Data Fixtures)

docker compose exec neuro-symbolic-extractor python src/seed_redis.py

4. Run the End-to-End Pipeline

docker compose exec neuro-symbolic-extractor python src/main.py 
```
## Roadmap for GSoC 2026
This prototype confirms the structural architecture. The next phase focuses on:

Full 50GB DBpedia Dump Integration: Moving from the seeded test fixtures to the complete Redis-backed DBpedia instance.

Bidirectional Predicate Inversion: Implementing advanced Spacy dependency parsing (nsubjpass) to automatically invert predicates during passive voice extraction.
