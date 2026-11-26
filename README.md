# Personal Profile RAG

A small **Retrieval-Augmented Generation** (RAG) service that answers questions about a personal profile. It runs fully locally: **ChromaDB** stores the embeddings, **Ollama** provides the embedding model and the LLM, and **FastAPI** exposes the API. Built as a learning project following the [NextWork RAG tutorial](https://nextwork.ai/projects/99ec4595-3d7b-5584-a799-aafb9c373bbb).

## How it works

1. **Index:** `build_knowledge_base.py` splits `profile.txt` into paragraph chunks, embeds each with `nomic-embed-text`, and stores them in a persistent ChromaDB collection (`personal_profile`).
2. **Retrieve:** `GET /ask` embeds the question and fetches the 2 most similar chunks, optionally filtered to one user via metadata.
3. **Augment:** the retrieved chunks are placed in a prompt along with the question.
4. **Generate:** the prompt is sent to a local `gemma2:9b` model via Ollama, and the answer is returned together with the context used.

## Tech stack

Python, FastAPI, Pydantic, ChromaDB, Ollama (`nomic-embed-text` for embeddings, `gemma2:9b` for generation).

## Getting started

### Prerequisites

- Python 3.10+
- [Ollama](https://ollama.com) running locally on port `11434`

### Setup

```bash
ollama pull nomic-embed-text
ollama pull gemma2:9b

python -m venv ragenv
source ragenv/bin/activate        # Windows: ragenv\Scripts\activate
pip install fastapi uvicorn chromadb ollama pydantic
```

### Build the knowledge base

Edit `profile.txt` (paragraphs separated by blank lines), then:

```bash
python build_knowledge_base.py
```

### Run the API

```bash
uvicorn main:app --reload
```

Interactive docs are at `http://localhost:8000/docs`.

## API

| Method | Path | Description |
| --- | --- | --- |
| `POST` | `/documents` | Add a profile. Body: `{"user_name": "...", "content": "..."}`. Content is chunked by paragraph and tagged with the user name. |
| `GET` | `/ask?question=...&user=...` | Answer a question from the stored chunks. `user` is optional and limits retrieval to that user's chunks. |

### Example

```bash
curl "http://localhost:8000/ask?question=What%20is%20my%20career%20goal%3F"
```

The response includes `answer`, `context_used` (the retrieved chunks) and `filtered_by_user`.

## Project structure

```
.
├── build_knowledge_base.py  # one-off indexing script
├── main.py                  # FastAPI app (ingest + ask)
└── profile.txt              # sample source document
```

`chroma_db/` (the vector store) and `ragenv/` are generated locally and git-ignored.

## Notes

`build_knowledge_base.py` uses fixed chunk IDs, so re-running it on an existing collection will fail on duplicate IDs. Delete `chroma_db/` to rebuild from scratch.
