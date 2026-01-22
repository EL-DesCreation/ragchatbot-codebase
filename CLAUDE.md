# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Run Commands

**Install dependencies:**
```bash
uv sync
```

**Run the application (use Git Bash on Windows):**
```bash
./run.sh
# or manually:
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Access points:**
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## Environment Setup

Copy `.env.example` to `.env` and set `ANTHROPIC_API_KEY`.

**Important:** Always use `uv` to run Python commands, never `pip` directly. This project uses `uv` for dependency management and execution.

## Architecture

This is a RAG (Retrieval-Augmented Generation) chatbot for querying course materials. The system uses Claude for AI responses and ChromaDB for vector storage.

### Request Flow

1. User query hits `/api/query` endpoint in `backend/app.py`
2. `RAGSystem` orchestrates the query processing
3. `AIGenerator` sends the query to Claude with tool definitions
4. Claude decides whether to use the `search_course_content` tool
5. If tool is used, `CourseSearchTool` queries `VectorStore` (ChromaDB)
6. `VectorStore` performs semantic search across two collections:
   - `course_catalog`: Course metadata for name resolution
   - `course_content`: Chunked course content for semantic search
7. Results flow back through Claude for response generation

### Key Components

- **RAGSystem** (`backend/rag_system.py`): Main orchestrator that wires together all components
- **VectorStore** (`backend/vector_store.py`): ChromaDB wrapper with two collections and unified search interface
- **AIGenerator** (`backend/ai_generator.py`): Claude API client with tool execution loop
- **DocumentProcessor** (`backend/document_processor.py`): Parses course documents and chunks text
- **ToolManager/CourseSearchTool** (`backend/search_tools.py`): Tool abstraction for Claude's tool use

### Data Models

Defined in `backend/models.py`:
- `Course`: Title, link, instructor, list of lessons
- `Lesson`: Number, title, link
- `CourseChunk`: Content chunk with course/lesson metadata for vector storage

### Document Format

Course documents in `docs/` follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [name]

Lesson 0: [lesson title]
Lesson Link: [url]
[lesson content...]

Lesson 1: [lesson title]
...
```

### Configuration

All settings in `backend/config.py`:
- `CHUNK_SIZE`: 800 characters per chunk
- `CHUNK_OVERLAP`: 100 character overlap between chunks
- `MAX_RESULTS`: 5 search results
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514

### Frontend

Static HTML/JS/CSS in `frontend/` served by FastAPI. The frontend communicates with `/api/query` and `/api/courses` endpoints.
