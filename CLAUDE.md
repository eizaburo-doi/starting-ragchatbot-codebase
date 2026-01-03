# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Important:** Always use `uv` to run Python commands and manage dependencies. Do not use `pip` directly.

```bash
# Install dependencies
uv sync

# Run the application (from root directory)
./run.sh
# Or manually:
cd backend && uv run uvicorn app:app --reload --port 8000

# Access points
# Web Interface: http://localhost:8000
# API Documentation: http://localhost:8000/docs
```

## Environment Setup

Requires a `.env` file in root with:
```
ANTHROPIC_API_KEY=your_key_here
```

## Architecture Overview

This is a RAG (Retrieval-Augmented Generation) chatbot for course materials. The system uses Claude as the AI backbone with tool-calling for semantic search.

### Request Flow

1. **User Query** → FastAPI endpoint (`/api/query`) → `RAGSystem.query()`
2. **RAGSystem** orchestrates: builds prompt, retrieves conversation history, calls AI with tools
3. **AIGenerator** sends request to Claude with `search_course_content` tool available
4. **Claude decides** whether to use the search tool based on the query
5. **If tool used** → `CourseSearchTool.execute()` → `VectorStore.search()` → results formatted and returned to Claude
6. **Claude generates** final response using search results (if any)
7. **Response** returned with sources list to frontend

### Core Components (backend/)

- **`rag_system.py`** - Main orchestrator connecting all components
- **`ai_generator.py`** - Claude API integration with tool handling loop
- **`vector_store.py`** - ChromaDB wrapper with two collections: `course_catalog` (metadata) and `course_content` (chunks)
- **`search_tools.py`** - Tool abstraction (`Tool` base class, `CourseSearchTool`, `ToolManager`)
- **`document_processor.py`** - Parses course documents into structured `Course`/`Lesson`/`CourseChunk` models
- **`session_manager.py`** - In-memory conversation history per session

### Document Format

Course documents in `docs/` follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: [title]
Lesson Link: [url]
[content...]

Lesson 1: [title]
[content...]
```

### Key Configuration (config.py)

- `CHUNK_SIZE`: 800 chars per chunk
- `CHUNK_OVERLAP`: 100 chars overlap
- `MAX_RESULTS`: 5 search results
- `MAX_HISTORY`: 2 conversation exchanges
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514

### Data Storage

- ChromaDB persists to `backend/chroma_db/`
- On startup, app loads documents from `docs/` folder (skips already-indexed courses)
