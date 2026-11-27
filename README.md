# 🧠 HiveMind — Distributed Multi-Agent AI Platform

**HiveMind** is a production-grade, microservices-based Multi-Agent AI Platform built on the MERN stack, LangGraph, and Redis. Unlike a standard single-LLM chatbot, HiveMind uses an intelligent **StateGraph Router** to dynamically analyze user prompts and route them to specialized AI agents (Chat, Real-Time Web Search, Coding, Document/Slide Generation, RAG, and Vision).

---

## 🏗️ System Architecture & How It Works

HiveMind follows a decoupled **Microservices Architecture** behind a centralized API Gateway:

```text
                       ┌──────────────────────────┐
                       │    React + Vite Client   │
                       │ (Tailwind + Redux + Auth)│
                       └────────────┬─────────────┘
                                    │ HTTP / Cookies
                                    ▼
                       ┌──────────────────────────┐
                       │    API Gateway (:8000)   │
                       │  (express-http-proxy +   │
                       │   Redis Auth Middleware) │
                       └─┬──────┬────────┬──────┬─┘
                         │      │        │      │
          ┌──────────────┘      │        │      └──────────────┐
          ▼                     ▼        ▼                     ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Auth Service    │  │   Chat Service   │  │  Agent Service   │  │ Billing Service  │
│     (:8001)      │  │     (:8002)      │  │     (:8003)      │  │   (Upcoming)     │
│ Firebase + Redis │  │ MongoDB Schemas  │  │ LangGraph Engine │  │ Razorpay Credits │
└────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘  └──────────────────┘
         │                     │                     │
         └─────────────────────┼─────────────────────┘
                               ▼
              ┌─────────────────────────────────┐
              │  MongoDB Atlas  |  Redis Cache  │
              └─────────────────────────────────┘
```

### 🔄 Request Flow
1. **Authentication:** Users sign in via Google (Firebase Client). The Auth Service verifies the token using Firebase Admin, creates/fetches the user in MongoDB, and stores a 7-day session in **Redis** via HTTP-only cookies.
2. **Gateway Routing:** Every request hits the **API Gateway**, which validates the Redis session via middleware and forwards the `x-user-id` custom header to downstream microservices.
3. **Multi-Agent Orchestration (LangGraph):** When a prompt hits the Agent Service, the **Router Node** evaluates the user's intent (or respects manual agent selection) and routes the state to the appropriate specialized agent.
4. **Conversational Memory:** Recent conversation history (sliding window of 20 messages) is cached in **Redis** per `conversationId` for low-latency context injection into LLM calls.

---

## ✨ Key Functionalities

* **Dynamic Agent Routing:** Powered by LangGraph `StateGraph` with conditional edges to automatically pick the best agent for the task.
* **Multi-Model LLM Support:** Flexible model configuration supporting **Groq** (for ultra-fast chat/routing) and **Google Gemini** (for complex reasoning/coding).
* **Real-Time Web Search:** Grounded internet search using **Tavily API** that returns structured answers along with relevant images and source links.
* **Persistent Multi-Conversation History:** ChatGPT-style sidebar with auto-generated conversation titles, MongoDB persistence, and Redis caching.
* **Rich Markdown & Code Rendering:** Supports GitHub-flavored Markdown (tables, lists, links), syntax-highlighted code blocks with one-click copy, and an interactive image lightbox.
* **Specialized AI Agents (Full Roadmap):**
  * 💬 **Chat Agent:** General Q&A, explanations, and context-aware conversations.
  * 🌐 **Search Agent:** Live web lookup with image & link extraction.
  * 💻 **Coding Agent:** Code generation, debugging, and live interactive HTML/CSS/JS previewer (Artifact panel).
  * 📄 **PDF & RAG Agent:** Generate downloadable PDFs and chat with uploaded PDF documents using Vector DB embeddings.
  * 📊 **PPT Agent:** Automated PowerPoint presentation generation.
  * 🎨 **Vision / Image Agent:** AI image generation and image analysis.
* **Credit & Billing System:** Plan-based credit limits (Free, Starter, Pro) integrated with Razorpay.

---

## 📊 Project Progress & Roadmap

### ✅ Phase 1: Core Working Prototype (7th Semester — Completed ~45%)
- [x] **Workspace & Infrastructure:** Microservices folder structure, Docker Compose setup for Redis, and shared `ioredis` client.
- [x] **API Gateway:** Request proxying (`express-http-proxy`), CORS, cookie parsing, and Redis-backed `protect` auth middleware.
- [x] **Auth Microservice:** Google OAuth via Firebase Admin SDK, MongoDB User schema, login/logout APIs, and Redis session management.
- [x] **Chat Microservice:** MongoDB schemas and CRUD REST APIs for Conversations and Messages.
- [x] **Core LangGraph Engine:** `AgentState` annotation, LLM Router node, Redis sliding-window conversational memory, **Chat Agent**, and **Tavily Web Search Agent**.
- [x] **Frontend Core UI:** React + Vite + Tailwind setup, Redux Toolkit state slices (`user`, `conversation`, `message`), collapsible Sidebar, Chat Area, Multi-Agent Input Bar, and Markdown/Syntax-Highlighted Message Bubbles.

### 🚧 Phase 2: Advanced Agents & Production Polish (8th Semester — Upcoming)
- [ ] **Coding Agent & Artifact UI:** Multi-file code generation with live split-screen previewer.
- [ ] **Document & Media Agents:** PDF generation, PPT generation, Image generation, and AWS S3 storage with presigned URLs.
- [ ] **Document RAG Pipeline:** File uploads via Multer, text chunking, Vector DB indexing, and PDF Q&A.
- [ ] **Billing Microservice:** Razorpay payment gateway integration and credit deduction middleware.
- [ ] **DevOps & Deployment:** Service-level Dockerfiles and GitHub Actions CI/CD deployment pipeline.

---

## 🛠️ Tech Stack

| Layer | Technologies |
| :--- | :--- |
| **Frontend** | React.js (Vite), Tailwind CSS, Redux Toolkit, Axios, React Markdown, Lucide React |
| **Backend** | Node.js, Express.js, `express-http-proxy`, Mongoose, Morgan |
| **AI / GenAI** | LangChain, LangGraph, Groq API, Google Gemini API, Tavily Search API |
| **Databases & Cache** | MongoDB Atlas, Redis (`ioredis` + Docker) |
| **Auth & Cloud** | Firebase Auth (Client & Admin SDK), AWS S3 (Phase 2), Razorpay (Phase 2) |

---

## 🚀 Local Setup & Running the Prototype

### Prerequisites
* Node.js (v18+)
* Docker Desktop (for running Redis)
* MongoDB Atlas URI, Firebase Project, Groq API Key, and Tavily API Key

### 1. Start Redis via Docker
```bash
cd backend
docker compose up -d
```

### 2. Configure Environment Variables
Create `.env` files in each service (`backend/gateway`, `backend/services/auth`, `backend/services/chat`, `backend/services/agent`, and `frontend`) with the required ports, `MONGODB_URI`, `REDIS_URL`, and API keys. Place your Firebase `serviceAccountKey.json` inside `backend/services/auth/`.

### 3. Run Microservices & Frontend
Open separate terminals for each service:
```bash
# Terminal 1 - API Gateway
cd backend/gateway && npm install && npm run dev

# Terminal 2 - Auth Service
cd backend/services/auth && npm install && npm run dev

# Terminal 3 - Chat Service
cd backend/services/chat && npm install && npm run dev

# Terminal 4 - Agent Service
cd backend/services/agent && npm install && npm run dev

# Terminal 5 - Frontend
cd frontend && npm install && npm run dev
```

---

## 👥 Team Members (7th Sem Major Project)

* **Lokendra Bhardwaj** *(Team Lead)* — System Architecture, Docker/Redis Setup, LangGraph Engine & Memory
* **Tarun** *(Backend)* — Auth Microservice (Firebase + Redis Sessions) & Billing Service
* **Aditya** *(Backend)* — API Gateway, Chat Microservice & Specialized AI Agents
* **Suraj** *(Frontend)* — React/Vite Setup, Redux Toolkit Store, Auth Flow & API Integration
* **Bhupendra** *(Frontend)* — Chat UI, Collapsible Sidebar, Markdown/Code Rendering & Artifact Panel
