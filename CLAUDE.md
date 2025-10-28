# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lorax is an AI-powered tool for analyzing and visualizing tree sequences using Large Language Models (LLMs). It uses Retrieval-Augmented Generation (RAG) to enable users to query tree-sequence data (tskit) using natural language, generating and executing Python code to answer those queries.

The system has two primary interfaces:
1. **Terminal chat interface** - Command-line interaction for tree-sequence analysis
2. **Web UI** - Flask + React application with real-time tree visualization

## Architecture

### Core Components

**LangGraph-Based Agent System** (Flow Engineering Approach):
- **Planner** ([graph.py:67-88](lorax/graph.py#L67-L88)) - Decomposes user queries into structured query plans using the `QueryPlan` model
- **Executor** ([graph.py:45-55](lorax/graph.py#L45-L55)) - Executes query plans by invoking appropriate tools
- **Generator** - Creates tskit code from natural language (with retry logic for failed executions)
- The workflow follows: `START -> planner -> executer -> generate -> END`

**Query Planning System** ([planner.py](lorax/planner.py)):
- `QueryPlan` - Container for structured queries with dependency graphs
- `Query` - Individual queries with tool type selection (VISUALIZATION, CODE_GENERATE, GENERAL_ANSWER, FETCH_FILE)
- Supports both single questions and multi-dependency queries that require merging responses
- Executes queries in dependency order (reverse topological sort)

**Tool System** ([tools.py](lorax/tools.py)):
- `generatorTool` - Generates and executes tskit code using RAG (retrieves relevant documentation via FAISS)
- `visualizationTool` - Generates Newick format tree visualizations
- `generalInfoTool` - Answers general questions using ReAct agent pattern
- `search_tskit` / `search_arxiv` - Search functions for documentation retrieval

**ReAct Agent** ([react_agent.py](lorax/react_agent.py)):
- Implements Reasoning + Acting pattern for complex queries
- Tools: `Name.TSKIT` (documentation search), `Name.ARXIV` (paper search)
- Max 5 iterations with structured JSON responses containing actions or answers
- Template-driven prompting from [prompts/react.txt](lorax/prompts/react.txt)

**Retrieval System** ([retriever/](lorax/retriever/)):
- FAISS vector store for tskit documentation
- Reranking with cross-encoder models for improved relevance
- Code parsing and chunking for documentation ingestion

### Prompt Management System

**CRITICAL: The Prompt class ([prompts.py](lorax/prompts.py)) has incomplete implementation**

The `Prompt` class is used throughout the codebase in three distinct patterns:

#### Pattern 1: LangChain Integration (JSON prompts)
```python
# graph.py:77-78, tools.py:54-55
prompt_messages = Prompt(agent_type='planner')  # or 'code_generation'
planner_prompt = ChatPromptTemplate.from_messages(prompt_messages)
```
- **JSON prompt files** (`planner.json`, `code_generation.json`) contain structured messages with `"role"` and `"content"` keys
- LangChain's `ChatPromptTemplate.from_messages()` expects an iterable of message tuples: `[("role", "content"), ...]`
- **ISSUE**: Prompt class is missing `__iter__()` method to convert JSON dicts to tuples

#### Pattern 2: String Concatenation (text prompts)
```python
# tools.py:39-40
prompt_messages = Prompt(agent_type='visualization')
question = prompt_messages + question
```
- **Text prompt files** (`visualization.txt`) are plain strings
- Code attempts to concatenate Prompt object with strings
- **ISSUE**: Prompt class is missing `__add__()`, `__radd__()`, and `__str__()` methods

#### Pattern 3: Direct File Reading (bypasses Prompt class)
```python
# react_agent.py:106
return read_file(PROMPT_TEMPLATE_PATH)  # Reads react.txt directly
```
- React agent bypasses the Prompt class entirely and reads the file with a utility function
- This works but is inconsistent with the rest of the codebase

**Prompt File Formats:**
- **JSON prompts**: Use LangChain's message format with placeholder variables like `{context}`, `{question}`, `{history}`, `{messages}`
- **Text prompts**: Plain text strings, may or may not use placeholders

**Path Resolution:**
- The Prompt class uses `_BASE_DIR = os.path.dirname(os.path.abspath(__file__))` to resolve paths relative to the `prompts.py` module location
- This ensures prompts can be found regardless of the current working directory
- Prompt files are located in `lorax/prompts/` directory

**Known Issues:**
1. Prompt class cannot be used with `ChatPromptTemplate.from_messages()` without `__iter__()` method
2. Prompt class cannot be concatenated with strings without `__add__()`, `__str__()` methods
3. Inconsistent usage patterns across the codebase
4. No unit tests for the Prompt class

### FastAPI Backend ([lorax_app.py](lorax/lorax_app.py))

- **File Upload** (`/api/upload`) - Accepts `.trees` files for analysis
- **Chat** (`/api/chat`) - Processes natural language queries, returns text responses
- **WebSocket** (`/ws/newick`) - Streams Newick visualization data to frontend
- **Conversation Memory** - Maintains chat history using LangChain's `ConversationBufferMemory`

### React Frontend ([lorax/website/taxonium_component/](lorax/website/taxonium_component/))

Built with:
- React 17, Vite build system
- deck.gl 8.6.6 for tree visualization
- MobX for state management
- Socket.io for real-time updates
- Tailwind CSS for styling

Key components in [src/components/](lorax/website/taxonium_component/src/components/)

### Model Abstraction ([models.py](lorax/models.py))

Unified interface supporting:
- **OpenAI**: gpt-4o, gpt-4, gpt-3.5-turbo
- **Ollama**: llama2, mistral, codellama

Default: `gpt-4o` (OpenAI)

## Common Development Commands

### Environment Setup

**IMPORTANT: This project has complex dependency resolution issues.**

The working `treesequences` conda environment has version conflicts that pip's dependency resolver catches when installing fresh. To reproduce the working environment:

```bash
conda create -n treesequences python=3.10
conda activate treesequences

# Install with --no-deps to bypass dependency checking
pip install -r requirements_clean.txt --no-deps
pip install -e . --no-deps
```

**OR use conda environment export (recommended for exact reproduction):**
```bash
# From working environment
conda env export > environment.yml

# To recreate
conda env create -f environment.yml
```

**Dependency Conflicts:**
- `langchain-tavily==0.2.12` requires `langchain>=0.3.20`
- Current requirements.txt pins `langchain==0.3.14`
- `aiohttp` version conflicts between different langchain packages
- These conflicts are ignored in the working environment due to installation order

### Running the Application

**Requires `.env` file with:**
```
OPENAI_API_KEY=your-api-key-here
```

**Web UI (default)**:
```bash
python launch.py
# Starts FastAPI server at http://localhost:8000
```

**Terminal chat interface**:
```bash
python lorax/langgraph_tskit.py
# Interactive CLI for tree-sequence queries
```

**Using the installed package**:
```bash
lorax
# Entry point defined in pyproject.toml
```

### Frontend Development
```bash
cd lorax/website/taxonium_component
yarn dev              # Development server
yarn build            # Production build
yarn storybook        # Component development
```

### Data Requirements

Tree-sequence data file must be placed in `lorax/data/` directory. Example data: [Google Drive link](https://drive.google.com/file/d/1pkV2PRwefiteQRreSd7DZsgkdyrmo1Wq/view?usp=drive_link)

## Key Implementation Details

### Code Execution Safety

Generated code is validated before execution ([utils.py:70-142](lorax/utils.py#L70-L142)):
- Import whitelist validation (only safe modules: numpy, tskit, msprime, stdlib)
- AST parsing for syntax checking
- Dynamic module execution with isolated namespace
- Automatic function discovery and parameter injection

### Structured Output

Uses Pydantic models for LLM structured outputs:
- `code` schema ([utils.py:40-46](lorax/utils.py#L40-L46)) - prefix, imports, code blocks
- `QueryPlan` schema - query graphs with dependencies
- Supports both OpenAI structured output API and Ollama response parsing

### Error Handling and Retries

Code generator has retry logic with max 3 iterations:
- Captures execution errors
- Feeds errors back to LLM for correction
- Terminates on success or max iterations ([graph.py:172-191](lorax/graph.py#L172-L191))

## Important Conventions

1. **File Paths**: Always use `attributes['file_path']` for tree-sequence data input
2. **Memory Management**: Pass `attributes['memory']` through all agent calls to maintain conversation context
3. **Tool Selection**: The planner automatically selects appropriate `ToolType` based on query classification
4. **Visualization Format**: Tree visualizations must return Newick strings for frontend rendering
5. **Code Structure**: Generated code must define a callable function (not just scripts) with `file_path` parameter

## Known Issues and Technical Debt

### Critical Issues

1. **Prompt Class Incomplete Implementation**
   - Missing `__iter__()` method - breaks `ChatPromptTemplate.from_messages()` usage
   - Missing `__add__()`, `__radd__()`, `__str__()` - breaks string concatenation in visualization tool
   - Inconsistent usage patterns across codebase (three different approaches)
   - Location: [prompts.py](lorax/prompts.py)
   - Affects: [graph.py:77-78](lorax/graph.py#L77-L78), [tools.py:39-40](lorax/tools.py#L39-L40), [tools.py:54-55](lorax/tools.py#L54-L55)

2. **Circular Import Fixed**
   - tools.py and react_agent.py had circular dependency
   - Fixed by moving import of `run` function inside `generalInfoTool()` function
   - Location: [tools.py:133](lorax/tools.py#L133)

3. **Dependency Version Conflicts**
   - requirements.txt contains incompatible version specifications
   - Working environment only works due to specific installation order
   - Affects fresh installations
   - Workaround: Use `pip install --no-deps` or `conda env export`

### Deprecation Warnings (non-critical)

1. **pkg_resources deprecated** ([tools.py:7](lorax/tools.py#L7))
   - Should migrate to `importlib.resources`
   - Slated for removal in Setuptools 81+

2. **LangChain FAISS import deprecated** ([retriever/create_documents.py:8](lorax/retriever/create_documents.py#L8))
   - Should use `from langchain_community.vectorstores import FAISS` instead of `from langchain.vectorstores import FAISS`

3. **ConversationBufferMemory deprecated** ([langgraph_tskit.py:10](lorax/langgraph_tskit.py#L10))
   - LangChain recommends migration to newer memory patterns
   - See: https://python.langchain.com/docs/versions/migrating_memory/

## Testing and Debugging

- Test queries: "Calculate the diversity of the given treesequence", "Show me the tree at position 1000"
- Check [pg.ipynb](pg.ipynb) and [lorax/test.ipynb](lorax/test.ipynb) for example usage
- Frontend components have Storybook stories in [src/stories/](lorax/website/taxonium_component/src/stories/)

## Build and Package

```bash
pip install -e .              # Editable install
python -m build               # Build distribution packages
```

Package metadata in [pyproject.toml](pyproject.toml) - version 0.1.0, Python 3.9-3.12 support.

## Troubleshooting

### "ModuleNotFoundError: No module named 'lorax'"
- Run `pip install -e .` from the project root
- Ensures the lorax package is installed in editable mode

### "FileNotFoundError: Prompt file not found"
- Fixed by making prompt paths relative to module location in prompts.py
- Paths now use `_BASE_DIR = os.path.dirname(os.path.abspath(__file__))`

### "TypeError: 'Prompt' object is not iterable"
- Known issue: Prompt class missing `__iter__()` method
- Affects usage with `ChatPromptTemplate.from_messages()`
- Occurs in graph.py (planner) and tools.py (code generator)

### Dependency conflicts during installation
- Use `pip install -r requirements_clean.txt --no-deps` to bypass checks
- Or use `conda env export` from working environment

### Code doesn't run from different directories
- Ensure you run `python lorax/langgraph_tskit.py` from project root
- Or use the `lorax` entry point after `pip install -e .`
