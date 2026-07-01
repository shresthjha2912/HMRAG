# HMRAG

**Hybrid Multimodal Retrieval-Augmented Generation for ScienceQA**

HMRAG is an end-to-end multimodal RAG pipeline designed for question answering over the ScienceQA dataset. It combines text, image understanding, vector retrieval, and graph-based reasoning to improve answer generation for multimodal science questions.

The system ingests ScienceQA questions, options, explanations, and associated images, builds a hybrid retrieval layer using **Qdrant** and **Neo4j**, and performs answer generation using hosted LLMs.

## Project Team

- 23CS30007
- 23CS30024
- 23CS10051

## Overview

Traditional RAG systems mainly retrieve text chunks using vector similarity. HMRAG extends this idea by combining:

- Multimodal understanding from images and question text
- Vector search using dense embeddings stored in Qdrant
- Knowledge graph reasoning using Neo4j
- Optional web retrieval
- LLM-based answer generation

This allows the system to retrieve both semantically similar chunks and structured graph context before generating the final answer.

## Repository Structure

```text
.
├── 23CS30007_23CS30024_23CS10051_HMRAG-2.pdf
├── README.md
├── hmrag_full.zip
└── hmrag_full_colab_mmkg_pushable.ipynb
```

| File | Description |
|---|---|
| `23CS30007_23CS30024_23CS10051_HMRAG-2.pdf` | Project report containing methodology, architecture, and results |
| `hmrag_full.zip` | Source package for the HM-RAG pipeline |
| `hmrag_full_colab_mmkg_pushable.ipynb` | Sanitized notebook for running ingestion and inference |
| `README.md` | Project documentation |

## Key Features

- End-to-end multimodal RAG pipeline
- ScienceQA dataset support
- Image-to-text description using vision-language models
- Dense vector retrieval with Qdrant
- Knowledge graph storage and retrieval with Neo4j
- Entity and relation extraction from multimodal context
- LLM-based answer generation
- Optional web retrieval
- Secure API key loading through environment variables or notebook secrets
- Notebook-based execution for easy reproducibility

## System Architecture

```text
ScienceQA Dataset
        |
        v
Question + Options + Image
        |
        v
Multimodal Preprocessing
        |
        v
Image Captioning / VLM Description
        |
        v
Chunking + Entity Extraction + Relation Extraction
        |
        +------------------------+
        |                        |
        v                        v
Qdrant Vector Store        Neo4j Knowledge Graph
        |                        |
        +-----------+------------+
                    |
                    v
Hybrid Retrieval
                    |
                    v
LLM-based Answer Generation
                    |
                    v
Final Answer
```

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python |
| Dataset | ScienceQA |
| Notebook Environment | Google Colab / Kaggle |
| Vector Database | Qdrant |
| Graph Database | Neo4j |
| LLM Provider | OpenAI-compatible hosted models |
| Vision-Language Model | SmolVLM / Qwen-VL style pipeline |
| Web Retrieval | Serper API |
| Embeddings | Text embedding models |
| Package Format | Jupyter Notebook + zipped Python package |

## Core Pipeline

### 1. Dataset Loading

The notebook downloads and prepares the ScienceQA dataset.

It locates:

- `problems.json`
- question splits such as `train`, `test`, and `val`
- associated image folders
- question metadata and answer options

### 2. Multimodal Ingestion

Each ScienceQA sample is converted into a multimodal context using:

- question text
- answer choices
- image path
- generated image description
- extracted entities
- extracted relations

### 3. Vector Store Construction

The processed chunks are embedded and inserted into Qdrant.

Qdrant is used for semantic similarity search during inference.

### 4. Knowledge Graph Construction

Neo4j is used to store structured information such as:

- questions
- chunks
- entities
- relations
- links between questions and extracted concepts

This helps the system retrieve context through graph relationships, not just vector similarity.

### 5. Hybrid Retrieval

During inference, HMRAG retrieves context from multiple sources:

- vector search from Qdrant
- graph context from Neo4j
- optional web context

The retrieved context is merged and passed to the answer generation model.

### 6. Answer Generation

A hosted LLM generates the final answer using the retrieved multimodal and graph-enhanced context.

The model returns the most likely option for the ScienceQA question.

## Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/shresthjha2912/HMRAG.git
cd HMRAG
```

### 2. Extract the Source Package

```bash
unzip hmrag_full.zip
```

This should create the main source directory:

```text
hmrag_full/
```

### 3. Install Dependencies

The notebook installs the required Python packages automatically. If running manually, install:

```bash
pip install -U openai qdrant-client neo4j tqdm requests==2.32.4 transformers accelerate qwen-vl-utils "Pillow<12.0"
```

## Required Secret Names

The project should load credentials only from environment variables, Colab secrets, or Kaggle secrets.

Configure the following secret names in the runtime environment:

```text
OPENAI_API_KEY
HF_TOKEN
QDRANT_HOST
QDRANT_PORT
QDRANT_API_KEY
QDRANT_COLLECTION
QDRANT_HTTPS
NEO4J_URI
NEO4J_USER
NEO4J_PASSWORD
SERPER_API_KEY
```

Do not write actual credential values inside notebooks, Python files, markdown files, or committed configuration files.

## Running the Notebook

Open the notebook:

```text
hmrag_full_colab_mmkg_pushable.ipynb
```

Run the cells in order.

The notebook performs:

1. dependency installation
2. ScienceQA download
3. source package setup
4. environment variable loading
5. dataset path detection
6. optional runtime compatibility fixes
7. multimodal ingestion
8. vector database insertion
9. graph database insertion
10. inference over ScienceQA samples

## Ingestion Command

The notebook internally runs a command similar to:

```bash
python -u -m hmrag_full.ingest_scienceqa_mmkg \
  --dataset "$PROBLEMS_JSON" \
  --image-root "$IMAGE_ROOT" \
  --split train \
  --max-items 500 \
  --qdrant-host "$QDRANT_HOST" \
  --qdrant-port "$QDRANT_PORT" \
  --qdrant-api-key "$QDRANT_API_KEY" \
  --qdrant-https \
  --neo4j-uri "$NEO4J_URI" \
  --neo4j-user "$NEO4J_USER" \
  --neo4j-password "$NEO4J_PASSWORD" \
  --openai-api-key "$OPENAI_API_KEY" \
  --embedding-model "text-embedding-3-small" \
  --use-vlm \
  --vlm-provider smolvlm \
  --vlm-model "HuggingFaceTB/SmolVLM-Instruct" \
  --kg-model "gpt-4o-mini"
```

## Security Guidelines

API keys and access tokens should never be committed to GitHub.

Avoid committing values for:

```text
OPENAI_API_KEY
HF_TOKEN
QDRANT_API_KEY
NEO4J_PASSWORD
SERPER_API_KEY
```

Recommended safety practices:

- Use Colab secrets, Kaggle secrets, or environment variables.
- Keep `.env` files out of Git.
- Add `.env`, `*.env`, and `.ipynb_checkpoints/` to `.gitignore`.
- Rotate any key that was accidentally committed.
- Run a local secret scan before pushing.

Example `.gitignore` entries:

```gitignore
.env
*.env
__pycache__/
.ipynb_checkpoints/
```

## Example Use Case

A ScienceQA question may contain:

- a text question
- multiple answer options
- an image
- supporting context

HMRAG converts this into a richer retrieval problem by extracting image descriptions, generating embeddings, storing graph relationships, and retrieving both vector and graph context before producing an answer.

## Advantages

- Combines semantic and structured retrieval
- Handles image-based question answering
- Supports graph-enhanced reasoning
- Uses hosted models instead of requiring local GPU LLM inference
- Can run in Colab or Kaggle environments
- Keeps secrets outside source code

## Limitations

- Requires external services such as Qdrant, Neo4j, and hosted LLM APIs
- Inference quality depends on retrieved context and model performance
- Large-scale ingestion may require more runtime and API quota
- The notebook is designed primarily for experimental and academic use

## Future Improvements

- Add a Streamlit or FastAPI interface
- Add automated evaluation over full ScienceQA splits
- Improve graph retrieval scoring
- Add support for local embedding models
- Add Docker-based setup
- Add experiment tracking and benchmark reports
- Add CI checks for notebook safety and secret scanning

## Report

The full project report is included in:

```text
23CS30007_23CS30024_23CS10051_HMRAG-2.pdf
```

It contains a more detailed explanation of the architecture, methodology, implementation, and experimental results.

## Summary

HMRAG demonstrates how multimodal retrieval, vector databases, knowledge graphs, and LLM-based reasoning can be combined into a single hybrid RAG system for visual question answering.

The project shows a practical pipeline for ingesting ScienceQA data, enriching it with multimodal and graph context, and generating answers using retrieved evidence.