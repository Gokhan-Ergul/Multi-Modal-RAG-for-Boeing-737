# Multi-Modal RAG for Boeing 737

This project is a **Retrieval-Augmented Generation (RAG) system** designed to answer technical aviation questions with **page-accurate citations** using the Boeing 737 Operations Manual.

Traditional PDF → text extraction fails on aviation manuals due to complex elements like multi-column layouts, tables, flowcharts, schematics, and special formatting.  
To solve this, the system uses a **Vision-First extraction pipeline**, converting pages into images and using **Gemini 2.5 Flash Vision** to structure the content into Markdown before embedding.

---

## Project Structure

### **1. `preprocessing.ipynb` – Data Ingestion & ETL**
Handles extraction + transformation of Boeing 737 manual pages.

**Pipeline includes:**

| Step | Description |
|------|-------------|
| PDF Cleaning | Removes headers/footers (e.g. dates, doc IDs) to prevent embedding noise. |
| Vision Extraction | Converts pages to images → LLM transcribes as Markdown (tables, diagrams, flow). |
| Intelligent Chunking | Manual splitting for dense pages + RecursiveCharacterTextSplitter. |
| Embedding | Uses **BAAI/bge-base-en-v1.5** for vector generation. |
| Storage | Stores vectors + metadata in **Qdrant local DB**. |

Pages 82–86 (performance tables) are manually split to avoid token overflow.

---

### **2. `Retrieval.py` – The RAG Engine**
Core script used for question answering.

**Features:**
- Loads Qdrant vector store + embedding model
- Retrieves top-K semantically relevant chunks
- Generates final answer via **Gemini 2.5 Flash**
- Requires **strict page citation for every claim**

**Functions to know:**
- `retrieve_similar_documents()` → vector lookup
- `format_context_for_llm()` → builds final answer prompt

---

### **3. `test.ipynb` – Evaluation Notebook**
Contains benchmark question-answer validation.

- Uses `boeing_737_test_set.csv`
- Runs full RAG inference
- Tracks:
  - **Page Score**: correct page retrieved?
  - **Content Score**: answer technically correct?

Performance is strong for **tables, diagrams, flowcharts**, but needs refinement in system component lookups (e.g. *Isolation Valve* queries).

---

## Prerequisites

| Requirement | Notes |
|------------|-------|
| Python 3.10+ | Required runtime |
| Poppler | Needed for `pdf2image` conversion → path must match your installation |
| Qdrant Local DB | Data stored under `./qdrant_db_data` |

Download Poppler → update path in `preprocessing.ipynb`:  
`C:\poppler-25.12.0\Library\bin`
