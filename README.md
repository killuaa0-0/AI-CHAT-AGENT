# AI Chat Agent

An AI-powered chat application that lets users upload PDF documents and have natural, context-aware conversations about their content. Built with Next.js and a Retrieval-Augmented Generation (RAG) pipeline backed by a vector database.

## Features

- 📄 **PDF Upload** — Upload one or more PDF documents through a simple drag-and-drop interface
- 💬 **Conversational Chat** — Ask questions in natural language and get answers grounded in your uploaded documents
- 🔍 **Vector Search (RAG)** — Documents are chunked and embedded, then retrieved semantically via Qdrant for accurate, context-aware responses
- ⚡ **Background Processing** — A dedicated worker handles document ingestion and embedding asynchronously, keeping the UI responsive
- 🔐 **Authentication** — User auth and session management handled by Clerk
- 🎨 **Modern UI** — Built with shadcn/ui components and Tailwind CSS

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15 (App Router), React 19 |
| Styling | Tailwind CSS 4, shadcn/ui |
| Auth | Clerk |
| Vector Database | Qdrant |
| Cache / Queue | Valkey (Redis-compatible) |
| Background Jobs | Node.js worker (`worker.js`) |
| Language | TypeScript / JavaScript |

## Architecture

```
┌──────────────┐     upload PDF     ┌──────────────┐
│   Next.js    │ ─────────────────▶ │   index.js   │
│   Frontend   │                    │  (API layer) │
└──────┬───────┘                    └──────┬───────┘
       │                                    │
       │ chat query                        │ enqueue job
       ▼                                    ▼
┌──────────────┐                    ┌──────────────┐
│    Qdrant    │ ◀── embeddings ─── │  worker.js   │
│ (vector DB)  │                    │ (processing) │
└──────────────┘                    └──────┬───────┘
                                            │
                                     ┌──────▼───────┐
                                     │    Valkey    │
                                     │ (cache/queue)│
                                     └──────────────┘
```

1. A user uploads a PDF via the **file-upload** component.
2. The document is queued and processed by `worker.js` — text is extracted, chunked, and embedded.
3. Embeddings are stored in **Qdrant** for semantic search; **Valkey** handles caching/queueing in between.
4. When the user asks a question in the **chat** interface, relevant chunks are retrieved from Qdrant and used to generate a grounded response.

## Prerequisites

- Node.js 18+
- [pnpm](https://pnpm.io/) (project uses `pnpm-lock.yaml`)
- [Docker](https://www.docker.com/) (for running Qdrant and Valkey locally)
- A Clerk account ([clerk.com](https://clerk.com)) for authentication keys

## Getting Started

1. **Clone the repository**
   ```bash
   git clone https://github.com/killuaa0-0/AI-CHAT-AGENT.git
   cd AI-CHAT-AGENT
   ```

2. **Install dependencies**
   ```bash
   pnpm install
   ```

3. **Start supporting services** (Qdrant + Valkey)
   ```bash
   docker compose up -d
   ```

4. **Configure environment variables**

   Create a `.env.local` file in the project root:
   ```env
   NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
   CLERK_SECRET_KEY=your_clerk_secret_key
   QDRANT_URL=http://localhost:6333
   VALKEY_URL=redis://localhost:6379
   # Add your LLM/embedding provider API key here, e.g.:
   # OPENAI_API_KEY=your_api_key
   ```

5. **Run the background worker**
   ```bash
   node worker.js
   ```

6. **Run the development server**
   ```bash
   pnpm dev
   ```

   Open [http://localhost:3000](http://localhost:3000) to view the app.

## Project Structure

```
├── chat.tsx           # Chat interface component
├── file-upload.tsx    # PDF upload component
├── button.tsx          # UI button component (shadcn/ui)
├── input.tsx            # UI input component (shadcn/ui)
├── layout.tsx           # App layout
├── page.tsx              # Main page
├── middleware.ts        # Clerk auth middleware
├── index.js             # API / server entry point
├── worker.js             # Background job processor (embedding, ingestion)
├── docker-compose.yml   # Qdrant + Valkey services
└── package.json
```

## License

This project is open source and available under the [MIT License](LICENSE).

---


