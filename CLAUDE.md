# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Start the full application (preferred method)
./run.sh

# Manual start (if needed)
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install/sync dependencies
uv sync

# Add new dependency
uv add <package-name>
```

### Environment Setup
- Create `.env` file with `ANTHROPIC_API_KEY=your_key_here`
- Place course documents in `/docs` folder for automatic loading

## Architecture Overview

### RAG System Flow
The application follows a tool-enabled RAG (Retrieval-Augmented Generation) pattern:

1. **Document Ingestion** (`document_processor.py`):
   - Processes PDF/DOCX/TXT files from `/docs` folder
   - Extracts structured course metadata (Course → Lessons → Chunks)
   - Chunks text into 800-character segments with 100-character overlap

2. **Vector Storage** (`vector_store.py`):
   - Uses ChromaDB with `all-MiniLM-L6-v2` embeddings
   - Maintains separate collections for course metadata and content chunks
   - Supports semantic search with course/lesson filtering

3. **AI Generation** (`ai_generator.py`):
   - Uses Anthropic Claude Sonnet 4 with tool-calling capabilities
   - Implements tool-based search rather than direct RAG context injection
   - Maintains conversation history through sessions

4. **Tool System** (`search_tools.py`):
   - Provides `search_course_content` tool for AI to query vector store
   - Supports course name and lesson number filtering
   - Tracks sources for response attribution

### Data Models (`models.py`)
```python
Course(title, course_link, instructor, lessons[])
Lesson(lesson_number, title, lesson_link)
CourseChunk(content, course_title, lesson_number, chunk_index)
```

### Key Integration Points

- **RAGSystem** (`rag_system.py:10`): Main orchestrator that coordinates all components
- **Session Management** (`session_manager.py`): Handles conversation context (2-message history)
- **FastAPI App** (`app.py:16`): Serves `/api/query` and `/api/courses` endpoints + static frontend
- **Frontend** (`frontend/script.js`): Vanilla JS chat interface with markdown support

### ChromaDB Collections
- `course_metadata`: Searchable course and lesson information
- `course_content`: Text chunks for content retrieval
- Storage path: `./chroma_db` (configurable in `config.py`)

### Tool-Based Search Pattern
Unlike traditional RAG that injects context directly, this system:
1. AI receives query and available tools
2. AI decides when to use `search_course_content` tool
3. Tool executes semantic search and returns formatted results
4. AI synthesizes response using tool results
5. Sources tracked separately for frontend display

### Configuration (`config.py`)
Key settings:
- `CHUNK_SIZE: 800` / `CHUNK_OVERLAP: 100`
- `MAX_RESULTS: 5` for search queries
- `MAX_HISTORY: 2` for conversation context
- Anthropic model: `claude-sonnet-4-20250514`

### Development Notes
- Documents auto-load from `/docs` on server startup (`app.py:88`)
- Frontend served as static files by FastAPI
- No build step required - direct file serving
- Environment variables loaded via `python-dotenv`
- make sure to use uv to manage all dependencies