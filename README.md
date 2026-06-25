# IRCTC AI Assistant - Multi-Agent Intent Router

## Overview

The IRCTC AI Assistant is an intelligent multi-agent conversational system designed to answer railway-related queries by routing each user request to the most appropriate specialized agent. Instead of relying on a single large language model for every query, the system first identifies the user's intent and then delegates the task to an expert agent, resulting in faster responses, lower API usage, and higher accuracy.

---

# System Architecture

```
                 User Query
                      │
                      ▼
            Intent Classification
                      │
         ┌────────────┼────────────┐
         │            │            │
         ▼            ▼            ▼
 Policy Agent   Booking Agent  General Agent
         │            │            │
         └────────────┼────────────┘
                      ▼
               Final Response
```

---

# Multi-Agent Workflow

## Step 1 — User Query

The user asks a question such as:

> "What is the refund policy?"

or

> "Book me a ticket from Delhi to Mumbai."

---

## Step 2 — Intent Router

The first component is the **Intent Router**, which determines:

* What the user wants
* Which agent should answer
* Confidence score for the prediction

Example:

```
User:
What is refund policy?

↓

Intent:
policy_query

↓

Confidence:
0.96

↓

Route:
Policy Agent
```

The router prevents unnecessary LLM calls by ensuring only the correct expert agent processes the request.

---

## Step 3 — Specialized Agent Selection

Depending on the predicted intent, the router forwards the request.

### Policy Agent

Handles:

* Refund Rules
* Cancellation Charges
* Tatkal Policy
* Senior Citizen Rules
* Insurance
* IRCTC Terms & Conditions
* Food Ordering
* Boarding Rules
* Railway Policies

This agent uses a Retrieval-Augmented Generation (RAG) pipeline.

---

### Booking Agent

Responsible for:

* Train Search
* Seat Availability
* Fare Queries
* Booking Assistance

---

### General Agent

Handles:

* Greetings
* Casual Conversation
* Questions outside the policy database
* Fallback responses

---

# Policy Agent (RAG Pipeline)

The Policy Agent uses Retrieval-Augmented Generation instead of directly asking Gemini for every question.

Pipeline:

```
User Question
      │
      ▼
Sentence Transformer
      │
      ▼
Query Embedding
      │
      ▼
FAISS Vector Search
      │
      ▼
Top Matching Policy Chunks
      │
      ▼
Confidence Evaluation
      │
 ┌────┴───────────────┐
 │                    │
 ▼                    ▼
High Confidence    Medium Confidence
 │                    │
 ▼                    ▼
Return Chunk      Gemini Summarization
 │                    │
 └────────────┬───────┘
              ▼
        Final Response
```

---

# Confidence-Based Decision Making

The Policy Agent uses cosine similarity scores to decide whether Gemini should be called.

### High Confidence

```
Similarity ≥ 0.80
```

* Returns the retrieved policy chunk directly.
* No Gemini API call.
* Lowest latency.
* Lowest cost.

---

### Medium Confidence

```
0.35 ≤ Similarity < 0.80
```

* Retrieves multiple relevant chunks.
* Sends them to Gemini.
* Gemini generates a concise answer strictly using retrieved context.

---

### Low Confidence

```
Similarity < 0.35
```

* No reliable policy found.
* Returns `None`.
* Router forwards the request to the General Agent.

---

# RAG Index Construction

During preprocessing:

1. PDFs are loaded.
2. Text is extracted using PyMuPDF.
3. Documents are split into chunks.
4. Chunks are embedded using Sentence Transformers.
5. Embeddings are stored inside a FAISS vector database.

This preprocessing happens only once.

At runtime, only vector retrieval is performed.

---

# Embedding Model

```
sentence-transformers/all-MiniLM-L6-v2
```

Dimension:

```
384
```

Used for:

* Document embeddings
* Query embeddings
* Semantic similarity search

---

# Vector Database

FAISS is used for efficient nearest-neighbor retrieval.

Stored files:

```
policy.index
chunks.pkl
```

These files are generated automatically after running:

```
python policy_rag.py --build
```

---

# Gemini Integration

Gemini is used only when retrieval confidence is moderate.

Advantages:

* Lower API cost
* Reduced hallucinations
* Faster responses
* Policy-grounded answers

If Gemini is unavailable, the system automatically falls back to returning the retrieved policy chunks.

---

# Features

* Multi-Agent Architecture
* Intent Classification
* Retrieval-Augmented Generation (RAG)
* FAISS Vector Search
* Sentence Transformer Embeddings
* Gemini-powered Summarization
* Confidence-based Routing
* Automatic Fallback Mechanism
* Modular Agent Design
* Local Knowledge Base

---

# Project Directory Structure

```
IRCTC-AI-Assistant/
│
├── front/
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── ...
│
├── back/
│
│   ├── agents/
│   │   ├── chatbot.py
│   │   ├── policy_rag.py
│   │   ├── intent_router.py
│   │   ├── general_agent.py
│   │   ├── booking_agent.py
│   │   └── ...
│   │
│   ├── data/
│   │   └── irctc_policies/
│   │       ├── raw_pdfs/
│   │       └── txt_fallbacks/
│   │
│   ├── vectorstore/
│   │   └── faiss_index/
│   │       ├── policy.index
│   │       └── chunks.pkl
│   │
│   ├── app.py
│   ├── requirements.txt
│   └── .env
│
├── Dockerfile
├── README.md
└── .gitignore
```

---

# Running the Project

## Backend

```
cd back
pip install -r requirements.txt
python app.py
```

or

```
uvicorn app:app --reload
```

---

## Frontend

```
cd front
npm install
npm run dev
```

---

# Environment Variables

Create a `.env` file inside the backend directory.

```
GEMINI_API_KEY=YOUR_API_KEY
```

---

# Technology Stack

* Python
* FastAPI
* React
* FAISS
* Sentence Transformers
* Google Gemini API
* PyMuPDF
* LangChain Text Splitter
* NumPy
* Docker

---
* Real-Time Booking Integration
* Analytics Dashboard
