# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend
uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Add new dependency
uv add package_name
```

### Environment Setup
```bash
# Copy environment template
cp .env.example .env
# Then edit .env to add your ANTHROPIC_API_KEY
```

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** built as a full-stack application for querying course materials using AI. The system uses a **tool-calling architecture** where Claude autonomously decides when to search course content vs. answer from general knowledge.

### Core Architecture Principles

**RAG System (rag_system.py)**: Main orchestrator that coordinates all components. Handles the complete query flow from user input to response generation.

**Tool-Calling Pattern**: The AI (Claude) has access to search tools and decides when to use them. This is implemented via:
- `search_tools.py`: Defines `CourseSearchTool` with Anthropic tool schema
- `ai_generator.py`: Handles tool execution and multi-turn conversations with Claude
- `tool_manager`: Orchestrates tool registration and execution

**Vector Search Pipeline**:
1. Documents processed into chunks via `document_processor.py`
2. Chunks embedded and stored in ChromaDB via `vector_store.py`
3. Semantic search performed when Claude calls the search tool
4. Results fed back to Claude for response generation

**Session Management**: Conversation context maintained across queries via `session_manager.py` with configurable history limits.

### Key Data Models (`models.py`)

- **Course**: Represents complete courses with metadata (title, instructor, lessons)
- **Lesson**: Individual lessons within courses with optional links
- **CourseChunk**: Text chunks for vector storage with course/lesson attribution

### Configuration (`config.py`)

All system parameters centralized:
- **Chunk Processing**: `CHUNK_SIZE=800`, `CHUNK_OVERLAP=100`
- **Search**: `MAX_RESULTS=5` chunks per search
- **AI Model**: Uses `claude-sonnet-4-20250514`
- **Embeddings**: Uses `all-MiniLM-L6-v2` sentence transformer
- **History**: `MAX_HISTORY=2` conversation turns

## Key Implementation Details

### Document Loading
- System auto-loads documents from `/docs` on startup (`app.py:startup_event`)
- Supports `.txt`, `.pdf`, `.docx` formats
- Prevents duplicate course processing by checking existing titles

### API Endpoints
- `POST /api/query`: Main query endpoint with session management
- `GET /api/courses`: Returns course statistics and titles
- Static file serving for frontend at root `/`

### Frontend Integration
- Vanilla JavaScript (`frontend/script.js`) with real-time UI updates
- Handles session management, loading states, and source attribution
- Uses fetch API for backend communication

### Error Handling
- Graceful fallbacks at each layer (document processing, vector search, AI generation)
- Frontend displays error messages for failed queries
- System continues operation even if individual documents fail to process

## Development Notes

- **No test framework**: This codebase has no test suite currently
- **Static frontend**: Pure HTML/CSS/JS, no build process required
- **ChromaDB storage**: Persistent vector database stored in `./chroma_db`
- **Development mode**: FastAPI runs with `--reload` for hot reloading
- **CORS**: Configured for development with wildcard origins

## Course Document Format

Documents in `/docs` should follow this structure:
```
Course Title: [Title]
Course Link: [Optional URL]
Course Instructor: [Optional Name]

Lesson [Number]: [Title]
Lesson Link: [Optional URL]
[Lesson content...]
```

The `document_processor.py` parses this format to extract structured course metadata and create searchable content chunks.
- always use use uv to run the server and manage all dependencies do not use pip directly