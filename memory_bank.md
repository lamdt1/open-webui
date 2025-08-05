# Open-WebUI Project Summary

This document provides a high-level overview of the Open-WebUI project, including its frontend and backend architecture, key features, and data models.

## Frontend (SvelteKit)

The frontend is built with SvelteKit and is located in the `src` directory.

### Routing

The application's routes are defined in the `src/routes` directory. Key routes include:

-   `(app)`: The main application layout, which includes the sidebar and content area. It contains sub-routes for various features like admin, chat, channels, home, notes, playground, and workspace.
-   `auth`: Handles user authentication.
-   `error`: Displays error pages.
-   `s`: Likely used for shared content.
-   `watch`: For watching video content.

### Libraries and Utilities

The `src/lib` directory contains the frontend's core logic and utilities:

-   `apis`: API clients for backend communication.
-   `components`: Reusable Svelte components.
-   `i18n`: Internationalization and localization.
-   `pyodide`: A Pyodide worker for running Python in the browser.
-   `stores`: Svelte stores for state management.
-   `types`: TypeScript type definitions.
-   `utils`: Utility functions.
-   `workers`: Web workers for background tasks.

## Backend (FastAPI)

The backend is a FastAPI application located in the `backend` directory.

### Application Structure

The main application logic is in `backend/open_webui`:

-   `main.py`: The main entry point for the FastAPI application, defining API routes and middleware.
-   `routers/`: API routers for different resources (e.g., auth, chats, models).
-   `models/`: SQLAlchemy database models.
-   `internal/`: Internal utilities, including database connection management.
-   `migrations/`: Alembic database migrations.
-   `retrieval/`: Information retrieval, including vector databases and web search.
-   `socket/`: Socket.IO server for real-time communication.
-   `storage/`: Storage provider for file uploads.
-   `utils/`: Backend utility functions.

### Key Features

-   **Chat Interface:** Allows users to interact with AI models.
-   **User Authentication:** Supports user signup and sign-in.
-   **File Uploads:** Enables users to upload files.
-   **Information Retrieval:** Can retrieve information from vector databases and web search.
-   **Real-time Communication:** Uses Socket.IO for real-time frontend-backend communication.
-   **Extensibility:** Designed to be extensible with custom functions and tools.

## Data Models

The backend uses SQLAlchemy for its database models, which are defined in `backend/open_webui/models`. Key models include:

-   `Auth`: User authentication data.
-   `Channel`: Communication channels.
-   `Chat`: Chat history and metadata.
-   `Feedback`: User feedback on messages.
-   `File`: Uploaded files.
-   `Folder`: Folders for organizing chats.
-   `Function`: Custom functions.
-   `Group`: User groups for access control.
-   `Knowledge`: Knowledge bases for retrieval.
-   `Memory`: User-specific memories.
-   `Message`: Individual messages within a chat.
-   `Model`: AI models.
-   `Note`: User notes.
-   `Prompt`: Reusable prompts.
-   `Tag`: Tags for organizing chats.
-   `Tool`: Custom tools.
-   `User`: User accounts.

## Infrastructure and Logic Flow Diagrams

### High-Level Infrastructure

```mermaid
graph TD
    subgraph "User's Browser"
        A[Frontend - SvelteKit]
    end

    subgraph "Backend Server"
        B[Backend - FastAPI]
        C[Database - PostgreSQL/SQLite]
        D[Vector DB - Chroma/etc.]
        E[WebSocket Server]
    end

    subgraph "External Services"
        F[Ollama Service]
        G[OAuth Providers]
        H[Web Search APIs]
    end

    A -- HTTP API Requests --> B
    A -- WebSocket Connection --> E
    B -- CRUD Operations --> C
    B -- Vector Search/Storage --> D
    B -- Forwards Requests --> F
    B -- Redirects/Callbacks --> G
    B -- API Calls --> H
    E -- Real-time Updates --> A
```

### Chat Completion Logic Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend
    participant WebSocket
    participant Database
    participant Ollama

    User->>Frontend: Enters and sends message
    Frontend->>Backend: POST /api/chat/completions (HTTP)
    Backend->>Database: Save user message
    Backend->>Ollama: Request model completion
    Ollama-->>Backend: Stream response chunks
    Backend->>WebSocket: Push chunks to user's channel
    WebSocket-->>Frontend: Receive stream chunks
    Frontend-->>User: Display response as it arrives
    Backend->>Database: Save full assistant response
```

### RAG (Web Search) Logic Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend
    participant WebSearchAPI
    participant VectorDB
    participant Ollama

    User->>Frontend: Sends message with web search enabled
    Frontend->>Backend: POST /api/chat/completions
    Backend->>Backend: Generate search queries
    Backend->>WebSearchAPI: Execute search queries
    WebSearchAPI-->>Backend: Return search results (links/snippets)
    Backend->>Backend: Scrape content from links
    Backend->>VectorDB: Store and index scraped content
    Backend->>VectorDB: Retrieve relevant context for user query
    Backend->>Ollama: Send user query + retrieved context
    Ollama-->>Backend: Generate response
    Backend-->>Frontend: Stream response to user
```