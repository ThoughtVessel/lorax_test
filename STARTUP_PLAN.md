# 🚀 Startup Plan: Understanding Lorax Database & RAG System

## **Phase 1: Core Database Understanding (Day 1-2)**

1. **Start with the Jupyter notebook** (`pg.ipynb`) - This is your goldmine! It contains:
   - Working examples of the RAG pipeline in action
   - FAISS vector database operations
   - tskit tree sequence queries
   - Code generation examples

2. **Key data structures to understand**:
   - **Tree Sequences**: `.trees` files (phylogenetic data using tskit library)
   - **Vector Database**: FAISS indices in `data/faiss-vector/` and multiple locations
   - **Documents**: Pickled documents in `data/documents.pkl` containing tskit documentation

3. **Run the existing system**:
   ```bash
   python launch.py  # Start the backend on localhost:8000
   cd frontend && yarn dev  # Start the frontend
   ```

## **Phase 2: RAG Pipeline Deep Dive (Day 2-3)**

1. **Study the retrieval system** in `lorax/chat/faiss_vector.py`:
   - Hybrid retrieval: BM25 + FAISS vector search
   - Cross-encoder reranking for relevance
   - Ensemble retriever combining multiple strategies

2. **Examine the code generation pipeline**:
   - `lorax/chat/tools.py` - Main tools including `generatorTool()`
   - `lorax/chat/utils.py` - Response parsing and code execution
   - Pattern: Query → Retrieve docs → Rerank → Generate code → Parse → Execute

3. **Test existing queries** from the notebook:
   ```python
   # Example query: "how many sites have 1 mutations"
   # Follow the full pipeline in pg.ipynb cells 5-22
   ```

## **Phase 3: Hands-On Experimentation (Day 3-4)**

1. **Load and explore sample data**:
   - Check if `data/sample.trees` exists, or find tree files in the system
   - Use the notebook to run queries on your own data
   - Understand tskit library basics for phylogenetic tree analysis

2. **Modify existing queries**:
   - Try different questions about tree sequences
   - Observe how the RAG system retrieves relevant documentation
   - Test code generation and execution pipeline

## **Phase 4: Architecture Understanding (Day 4-5)**

1. **WebSocket integration** (`lorax/lorax_app.py`, `lorax/manager.py`):
   - Real-time communication between frontend and backend
   - Chat interface for RAG queries
   - Visualization updates

2. **Frontend integration** (`frontend/` - React/Vite):
   - Taxonium component for phylogenetic visualization
   - Chat interface consuming RAG responses

## **🎯 Quick Start Actions (First 2 Hours)**

1. **Install dependencies**: `pip install -r requirements.txt`
2. **Open the notebook**: `jupyter notebook pg.ipynb`
3. **Run cells 1-22** to see the full RAG pipeline in action
4. **Check existing vector databases**: `ls -la data/faiss-vector/`
5. **Try a simple query**: Use the `generatorTool()` function with your own question

## **Key Files to Focus On**

- `pg.ipynb` - Your learning playground with working examples
- `lorax/chat/tools.py` - Main RAG tools and code generation
- `lorax/chat/faiss_vector.py` - Vector database and retrieval logic
- `lorax/handlers.py` - File upload and tree sequence processing
- `data/` directory - Sample data and vector indices

## **Notes**

This focused approach will get you productive quickly by starting with working examples and building up your understanding incrementally. The notebook is particularly valuable as it shows the complete RAG pipeline working end-to-end.

The system combines:
- **tskit**: Tree sequence analysis library for phylogenetic data
- **FAISS**: Vector similarity search for document retrieval
- **LangChain**: RAG pipeline orchestration
- **Ollama/OpenAI**: LLM backends for code generation
- **React**: Frontend visualization and chat interface