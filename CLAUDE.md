# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lorax is a visualization framework for Ancestral Recombination Graphs built with a Python FastAPI backend and React frontend. The project combines phylogenetic tree visualization with chat-based interaction capabilities.

## Architecture

### Backend (Python)
- **Main application**: `lorax/lorax_app.py` - FastAPI application with WebSocket support
- **Entry point**: `launch.py` - Launches the main interface via `lorax.interface.main()`
- **Core components**:
  - `lorax/handlers.py` - LoraxHandler for file processing
  - `lorax/manager.py` - WebSocketManager for real-time communication
  - `lorax/chat/` - Chat functionality with planner and React-based tools
  - `lorax/chat/react_from_scratch/` - Chat implementation with LLM integration

### Frontend (React/Vite)
- **Location**: `frontend/` directory
- **Framework**: React with Vite build system, uses Yarn for package management
- **Component library**: "taxonium-component" for phylogenetic tree exploration
- **Key libraries**: deck.gl for visualization, React Router for navigation

### Development Structure
- **Data**: `data/` - Sample data files
- **Configuration**: Docker setup with Nginx reverse proxy
- **CI/CD**: GitHub Actions workflow for frontend builds (`.github/workflows/build.yml`)

## Common Development Commands

### Frontend Development
```bash
cd frontend
yarn install          # Install dependencies
yarn dev              # Start development server
yarn build            # Build for production
yarn lint             # Run ESLint
```

### Backend Development
```bash
pip install -r requirements.txt    # Install Python dependencies
python launch.py                   # Start the development server (runs on localhost:8000)
```

### Docker
```bash
docker build -t lorax .    # Build Docker image
docker run -it -p 80:80 lorax    # Run containerized application
```

## Key Technologies

- **Backend**: FastAPI, uvicorn, WebSockets, SQLAlchemy, pandas, scikit-learn
- **Frontend**: React 18, Vite, TailwindCSS, deck.gl, d3-scale
- **Data Processing**: tskit (tree sequence toolkit), numpy, faiss-cpu
- **Testing**: Python unittest framework (found in `lorax/chat/react_from_scratch/src/code-chunker/tests/`)

## WebSocket Communication

The application uses WebSockets for real-time communication between frontend and backend via `WebSocketManager`. Components can send messages to specific channels like "viz" for visualization updates.

## File Upload Handling

Files are uploaded to `uploads/` directory and processed by `LoraxHandler`, which generates both visualization and chat configurations for phylogenetic data.