# 🌊 FloatChat - ARGO RAG System

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green.svg)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-12+-blue.svg)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)
![Status](https://img.shields.io/badge/Status-Active-success.svg)

**A Retrieval-Augmented Generation (RAG) system for querying ARGO oceanographic data using natural language**

---

## 🎯 Overview

**FloatChat** is an intelligent RAG (Retrieval-Augmented Generation) system designed to make ARGO oceanographic data accessible through natural language queries. It combines vector similarity search with LLM-powered SQL generation to enable users to ask questions like *"What's the average salinity in 2025?"* and receive accurate, data-driven answers.

### What it does

- **Natural Language Interface**: Ask questions about ARGO oceanographic data in plain English
- **Intelligent Retrieval**: Uses pgvector embeddings to find relevant context from the database
- **SQL Generation**: Leverages Google Gemini to generate safe, read-only SQL queries
- **Rich Visualizations**: Displays results with interactive tables, charts, and maps
- **Session Management**: MCP (Model Context Protocol) integration for conversation context

---

## ✨ Features

### Core Capabilities

- 🔍 **Semantic Search**: Vector-based similarity search using pgvector and sentence-transformers
- 🤖 **LLM-Powered SQL**: Google Gemini generates safe, validated SQL queries from natural language
- 🗺️ **Interactive Visualizations**: 
  - Data tables with filtering and sorting
  - Depth profiles and histograms
  - Geographic maps with lat/lon detection
- 📊 **Multi-Year Support**: Query data across years (2001-2017) with automatic UNION ALL handling
- 🔒 **SQL Safety**: Built-in guards prevent malicious SQL execution (read-only queries)
- 💾 **Resumable Ingestion**: Process NetCDF files incrementally without duplicates
- 🎨 **Modern UI**: Beautiful Dash frontend with dark/light mode support

### Data Processing

- **NetCDF Ingestion**: Batch processing of oceanographic NetCDF files
- **Embedding Generation**: Automatic summarization and vectorization of schema and data
- **Idempotent Updates**: Resume interrupted ingestion processes seamlessly

---

## 🛠️ Tech Stack

### Backend
- **FastAPI** (0.100+) - Modern Python web framework
- **PostgreSQL** (12+) - Relational database
- **pgvector** (0.4.1) - Vector similarity search extension
- **Sentence-Transformers** (2.2.2) - Text embeddings (`all-MiniLM-L6-v2`)
- **Google GenAI** (1.24.0) - Gemini 2.0 Flash for SQL generation
- **psycopg2** (2.9.9) - PostgreSQL adapter

### Frontend
- **Dash** (2.17+) - Interactive web application framework
- **Dash Bootstrap Components** (1.5+) - UI components
- **Plotly** (5.22+) - Interactive visualizations
- **Pandas** (2.2.3) - Data manipulation

### Data Processing
- **netCDF4** - NetCDF file handling
- **NumPy** (1.26+) - Numerical operations

---

### Data Flow

1. **User Query** → Frontend sends natural language question
2. **Embedding** → Question is vectorized using sentence-transformers
3. **Retrieval** → pgvector finds top-k similar context from `argo_embeddings`
4. **SQL Generation** → Gemini generates SQL query with retrieved context + schema
5. **Validation** → SQL guard validates query safety and syntax
6. **Execution** → Query runs against PostgreSQL (read-only)
7. **Visualization** → Results displayed in tables, charts, and maps

---

## 📦 Prerequisites

- **Python** 3.8 or higher
- **PostgreSQL** 12+ with pgvector extension
- **Google Gemini API Key** ([Get one here](https://makersuite.google.com/app/apikey))
- **NetCDF files** (optional, for data ingestion)

---

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/argo_rag.git
cd argo_rag
```

### 2. Create Virtual Environment

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux/Mac
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Set Up PostgreSQL with pgvector

```sql
-- Connect to PostgreSQL
CREATE DATABASE argo_db;
\c argo_db

-- Install pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create tables (schema will be created by ingestion scripts)
```

## 🎮 Usage

### 1. Build Vector Embeddings

First, ingest your data and build embeddings:

```bash
# Set database connection
export PG_DSN=postgresql://user:password@localhost:5432/argo_db

# Build embeddings for schema and rows
python -m scripts.build_pgvector_embeddings --ingest schema rows

# Or with limits
python -m scripts.build_pgvector_embeddings --ingest schema rows --max-rows 100000

# Resume interrupted ingestion (skips existing)
python -m scripts.build_pgvector_embeddings --ingest rows --max-rows 200000
```

### 2. Start Backend Server

```bash
# Development mode with auto-reload
uvicorn backend.app.main:app --host 0.0.0.0 --port 8080 --reload

# Production mode
uvicorn backend.app.main:app --host 0.0.0.0 --port 8080
```

The API will be available at `http://localhost:8080`

- API Docs: `http://localhost:8080/docs` (Swagger UI)
- Alternative Docs: `http://localhost:8080/redoc` (ReDoc)

### 3. Start Frontend

```bash
# Using Dash app v2 (recommended)
python frontend/dash_app_v2.py

# Or original Dash app
python frontend/dash_app.py

# Or Streamlit app (alternative)
streamlit run streamlit_app/app.py
```

The frontend will be available at `http://localhost:8501`

## 📁 Project Structure

```text
argo_rag/
├── backend/
│   └── app/
│       ├── __init__.py
│       ├── main.py              # FastAPI application
│       ├── config.py            # Configuration management
│       ├── db.py                # Database connection utilities
│       ├── llm/
│       │   └── gemini.py        # Google Gemini integration
│       ├── mcp/
│       │   ├── models.py        # MCP protocol models
│       │   ├── router.py        # MCP endpoints
│       │   └── session.py       # Session management
│       ├── rag/
│       │   ├── context.py       # Context building
│       │   ├── retrieval.py     # Vector similarity search
│       │   └── sql_guard.py     # SQL safety validation
│       └── routes/
│           ├── health.py        # Health check endpoint
│           └── qa.py            # Query endpoint
├── frontend/
│   ├── dash_app.py              # Original Dash frontend
│   └── dash_app_v2.py           # Enhanced Dash frontend (recommended)
├── ingestion_layer/
│   └── ingestion_1.py           # NetCDF to PostgreSQL ingestion
├── scripts/
│   ├── build_pgvector_embeddings.py  # Embedding generation script
│   └── vector_embeddings_v2.py       # Alternative embedding script
├── streamlit_app/
│   └── app.py                   # Streamlit alternative frontend
├── legacy/                      # Legacy code (for reference)
├── .env                         # Environment variables (create this)
├── .gitignore
├── requirements.txt             # Python dependencies
└── README.md                    # This file
```

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---

## 📺 Demo & Resources

### Demo Video

https://github.com/user-attachments/assets/73d9a059-98cd-41fa-8c57-bc72353f7537

### Live Deployment
[soon]


---

## 🔮 Future Enhancements

- [ ] Multi-table JOIN support
- [ ] Caching layer for frequent queries
- [ ] Export to multiple formats (JSON, Excel)
- [ ] User authentication and query history
- [ ] Advanced chart types (3D visualizations)
- [ ] Real-time data updates
- [ ] Multi-language support

---

## 👥 Authors

**Raghav Tiwari**
- B.Tech Computer Science Engineering
- Software Engineering | Data Analytics | Machine Learning | Cloud
