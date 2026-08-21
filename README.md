# 📚 Library Assistant

A Retrieval-Augmented Generation (RAG) chatbot that answers questions about your library's policies, rules, and FAQs — grounded entirely in your own PDF documents. Built with Streamlit, local embeddings, and Groq for fast LLM inference.

## Features

- **Upload & index PDFs** — library handbooks, policies, rules, notices, FAQs, and more.
- **Local, free embeddings** — uses `sentence-transformers/all-MiniLM-L6-v2` via HuggingFace, so no embedding API key is required.
- **Lightweight vector store** — a simple NumPy-based cosine-similarity store (no compiled dependencies like FAISS, so it installs cleanly on any Python version).
- **Fast generation via Groq** — retrieved chunks are passed to a Groq-hosted model for grounded, low-latency answers.
- **Source-cited answers** — every response can be traced back to the original document and page.
- **Guardrailed responses** — the assistant only answers from the provided context and won't invent policies, deadlines, or numbers.

## How It Works

1. **Load** — PDF documents are loaded and parsed page-by-page.
2. **Split** — text is split into overlapping chunks (1000 chars, 150 overlap) for better retrieval granularity.
3. **Embed** — each chunk is embedded locally using a HuggingFace sentence-transformer.
4. **Store** — embeddings are normalized and stored in an in-memory/on-disk NumPy vector store.
5. **Retrieve** — on a user question, the top-k most similar chunks are retrieved via cosine similarity.
6. **Generate** — the question plus retrieved context is sent to a Groq-hosted LLM, which produces a concise, grounded answer.

## Project Structure

```
.
├── app.py           # Streamlit UI (upload, index, chat)
├── rag_core.py       # Core RAG pipeline: loading, splitting, embedding, retrieval, generation
├── faiss_index/       # Persisted vector store (created after indexing)
└── README.md
```

## Setup

### 1. Clone the repo

```bash
git clone <your-repo-url>
cd <your-repo-folder>
```

### 2. Install dependencies

```bash
pip install streamlit langchain-community langchain-text-splitters langchain-huggingface langchain-groq numpy pypdf
```

### 3. Add your Groq API key

Get a free key from [console.groq.com](https://console.groq.com/keys), then either:

**Option A — Streamlit secrets** (recommended for Streamlit Cloud):

Create `.streamlit/secrets.toml`:

```toml
GROQ_API_KEY = "your-key-here"
```

**Option B — Environment variable:**

```bash
export GROQ_API_KEY="your-key-here"
```

### 4. Run the app

```bash
streamlit run app.py
```

## Configuration

Key settings live at the top of `rag_core.py`:

| Setting | Default | Description |
|---|---|---|
| `EMBEDDING_MODEL` | `sentence-transformers/all-MiniLM-L6-v2` | Local HuggingFace embedding model |
| `OLLAMA_MODEL` | `openai/gpt-oss-120b` | Groq-hosted generation model |
| `CHUNK_SIZE` | `1000` | Characters per chunk |
| `CHUNK_OVERLAP` | `150` | Overlap between chunks |
| `INDEX_DIR` | `faiss_index` | Where the vector store is persisted |

## Usage

1. Upload your library's PDFs (policies, rules, FAQs, notices) from the sidebar.
2. Wait for indexing to finish — you'll see a chunk/doc count once done.
3. Ask questions in plain language, e.g. *"What's the fine for a late book return?"* or *"How many books can a student borrow at once?"*
4. The assistant answers using only the indexed documents, and points you to the admin/library office if the answer isn't in the docs.

## Troubleshooting

**`groq.NotFoundError` / `model_not_found`**
The model set in `OLLAMA_MODEL` has been decommissioned by Groq. Check [Groq's supported models](https://console.groq.com/docs/models) and update the value in `rag_core.py` (e.g. to `openai/gpt-oss-120b` or `openai/gpt-oss-20b`).

**`GROQ_API_KEY is not set`**
Add your key to `.streamlit/secrets.toml` or as an environment variable (see Setup step 3).

**Slow first run**
The first query downloads and loads the local embedding model — subsequent runs are much faster.

## Tech Stack

- [Streamlit](https://streamlit.io/) — UI
- [LangChain](https://www.langchain.com/) — document loading & text splitting
- [HuggingFace sentence-transformers](https://www.sbert.net/) — local embeddings
- [Groq](https://groq.com/) — LLM inference
- NumPy — vector similarity search

## License

Add your license here.
