## hello-world1

> > This file contains essential context for Claude to understand the project when starting new chat sessions.

# 2nd Brain - Claude Context File

> This file contains essential context for Claude to understand the project when starting new chat sessions.

---

## Product Overview

**2nd Brain** is an **AI-powered knowledge transfer system** for enterprises/organizations. It preserves organizational knowledge by ingesting emails, documents, and messages, making them searchable via AI, and identifying knowledge gaps.

### Core Problem Solved
When employees leave or knowledge is siloed, organizations lose critical information. 2nd Brain captures everything, makes it searchable, identifies what's missing, and helps transfer knowledge to new team members.

---

## Architecture (V2 - B2B SaaS)

```
Frontend (Next.js 14)              Backend (Flask)
├── Chat Interface          →      ├── Auth API (JWT + bcrypt)
├── Documents Page          →      ├── Integration API (Gmail/Slack/Box)
├── Knowledge Gaps          →      ├── Document API (Classification)
├── Projects View           →      ├── Knowledge API (Gaps + Whisper)
├── Integrations            →      ├── Video API (Generation)
└── Training Guides         →      └── RAG Engine

Port 3000/3006 (frontend)          Port 5003 (backend)
```

### Directory Structure
```
/Users/rishitjain/Downloads/2nd-brain/
├── frontend/                    # Next.js 14 app
│   ├── app/                     # Pages
│   ├── components/              # React components
│   └── lib/                     # Utilities, auth context
├── backend/
│   ├── app_v2.py               # NEW: Main Flask app (V2 with all features)
│   ├── app_universal.py        # Legacy Flask app
│   ├── api/                    # NEW: API blueprints
│   │   ├── auth_routes.py      # Authentication endpoints
│   │   ├── integration_routes.py # OAuth & sync endpoints
│   │   ├── document_routes.py  # Classification endpoints
│   │   ├── knowledge_routes.py # Gaps & transcription
│   │   └── video_routes.py     # Video generation
│   ├── database/               # NEW: Database layer
│   │   ├── config.py           # DB configuration
│   │   └── models.py           # SQLAlchemy models
│   ├── services/               # NEW: Business logic
│   │   ├── auth_service.py     # JWT, bcrypt, sessions
│   │   ├── classification_service.py # AI classification
│   │   ├── knowledge_service.py # Gaps, Whisper, embeddings
│   │   └── video_service.py    # Video generation
│   ├── connectors/             # Integration connectors
│   │   ├── gmail_connector.py  # Gmail OAuth
│   │   ├── slack_connector.py  # Slack OAuth
│   │   └── box_connector.py    # NEW: Box OAuth
│   ├── rag/                    # RAG engine
│   │   └── enhanced_rag_v2.py  # Primary RAG
│   ├── club_data/              # BEAT Club dataset
│   ├── data/                   # Enron dataset
│   └── tenant_data/            # NEW: Per-tenant data directories
```

---

## B2B SaaS User Flow (IMPLEMENTED)

```
New User Signs Up (no data)     → POST /api/auth/signup
       ↓
Connect Slack + Box + Gmail     → GET /api/integrations/{type}/auth
       ↓
Ingest & Parse all data         → POST /api/integrations/{type}/sync
       ↓
Classify Work vs Personal (AI)  → POST /api/documents/classify
       ↓
User Reviews/Confirms           → POST /api/documents/{id}/confirm
       ↓
Build Knowledge Base            → POST /api/knowledge/rebuild-index
       ↓
Identify Knowledge Gaps         → POST /api/knowledge/analyze
       ↓
RAG Search + Video Generation   → POST /api/search, POST /api/videos
```

---

## Feature Status (POST-BUILD)

| Feature | Status | Completeness |
|---------|--------|--------------|
| User Signup/Login | 🟢 COMPLETE | 100% |
| Gmail Integration | 🟢 COMPLETE | 100% |
| Slack Integration | 🟢 COMPLETE | 100% |
| Box Integration | 🟢 COMPLETE | 100% |
| Document Classification | 🟢 COMPLETE | 100% |
| User Review/Confirm | 🟢 COMPLETE | 100% |
| Knowledge Gaps | 🟢 COMPLETE | 100% |
| Answer Persistence | 🟢 COMPLETE | 100% |
| Whisper Transcription | 🟢 COMPLETE | 100% |
| Index Rebuild | 🟢 COMPLETE | 100% |
| RAG Search | 🟢 COMPLETE | 100% |
| Video Generation | 🟢 COMPLETE | 100% |
| Multi-Tenant | 🟢 COMPLETE | 100% |

---

## Technology Stack

### Backend
- **Framework**: Flask 3.0
- **Database**: SQLAlchemy 2.0 (SQLite dev / PostgreSQL prod)
- **Auth**: PyJWT + bcrypt
- **AI**: Azure OpenAI (GPT-5, text-embedding-3-large, Whisper)
- **TTS**: Azure Cognitive Services Speech
- **Video**: MoviePy + PIL

### Frontend
- **Framework**: Next.js 14, React 18, TypeScript
- **Styling**: Tailwind CSS
- **HTTP**: Axios

### Integrations
- **Gmail**: google-auth, google-api-python-client
- **Slack**: slack-sdk
- **Box**: boxsdk

---

## Database Models

| Model | Purpose |
|-------|---------|
| `Tenant` | Organization/company (multi-tenant isolation) |
| `User` | User accounts with bcrypt passwords |
| `UserSession` | JWT refresh tokens, session management |
| `Connector` | Integration configs (Gmail/Slack/Box) |
| `Document` | Ingested content with classification |
| `DocumentChunk` | Embedding chunks for RAG |
| `Project` | Topic clusters |
| `KnowledgeGap` | Identified gaps with questions |
| `GapAnswer` | User answers (text or voice) |
| `Video` | Generated training videos |
| `AuditLog` | Action audit trail |

---

## API Endpoints (V2)

### Authentication
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/signup` | POST | Register new user + organization |
| `/api/auth/login` | POST | Login with email/password |
| `/api/auth/logout` | POST | Logout current session |
| `/api/auth/refresh` | POST | Refresh access token |
| `/api/auth/me` | GET | Get current user info |

### Integrations
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/integrations` | GET | List all integrations |
| `/api/integrations/{type}/auth` | GET | Start OAuth flow |
| `/api/integrations/{type}/callback` | GET | OAuth callback |
| `/api/integrations/{type}/sync` | POST | Trigger sync |
| `/api/integrations/{type}/disconnect` | POST | Disconnect |

### Documents
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/documents` | GET | List documents with filters |
| `/api/documents/classify` | POST | Classify pending documents |
| `/api/documents/{id}/confirm` | POST | Confirm classification |
| `/api/documents/{id}/reject` | POST | Reject as personal |
| `/api/documents/bulk/confirm` | POST | Bulk confirm |
| `/api/documents/stats` | GET | Classification statistics |

### Knowledge Gaps
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/knowledge/analyze` | POST | Analyze docs for gaps |
| `/api/knowledge/gaps` | GET | List knowledge gaps |
| `/api/knowledge/gaps/{id}/answers` | POST | Submit answer |
| `/api/knowledge/transcribe` | POST | Whisper transcription |
| `/api/knowledge/gaps/{id}/voice-answer` | POST | Voice answer |
| `/api/knowledge/rebuild-index` | POST | Rebuild embeddings |

### Videos
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/videos` | POST | Create video |
| `/api/videos` | GET | List videos |
| `/api/videos/{id}` | GET | Get video details |
| `/api/videos/{id}/status` | GET | Get generation progress |
| `/api/videos/{id}/download` | GET | Download video file |

### Search
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/search` | POST | RAG search with AI answer |
| `/api/health` | GET | Health check |

---

## Running the Application (V2)

```bash
# Terminal 1 - Backend (V2)
cd /Users/rishitjain/Downloads/2nd-brain/backend
export AZURE_OPENAI_API_KEY="your-key-here"
./venv_new/bin/python app_v2.py
# Runs on http://localhost:5003

# Terminal 2 - Frontend
cd /Users/rishitjain/Downloads/2nd-brain/frontend
npm run dev -- -p 3006
# Runs on http://localhost:3006
```

### Environment Variables
```bash
# Required
AZURE_OPENAI_API_KEY=your-key

# Optional (for integrations)
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
SLACK_CLIENT_ID=...
SLACK_CLIENT_SECRET=...
BOX_CLIENT_ID=...
BOX_CLIENT_SECRET=...

# Optional (for Azure TTS)
AZURE_TTS_KEY=...
AZURE_TTS_REGION=eastus2
```

---

## Key Implementation Files (NEW)

| File | Purpose |
|------|---------|
| `backend/app_v2.py` | Main Flask app with all blueprints |
| `backend/database/models.py` | SQLAlchemy ORM models |
| `backend/services/auth_service.py` | JWT + bcrypt authentication |
| `backend/services/classification_service.py` | GPT-4 work/personal classifier |
| `backend/services/knowledge_service.py` | Gaps, Whisper, embeddings |
| `backend/services/video_service.py` | Video generation pipeline |
| `backend/connectors/box_connector.py` | Box OAuth + file sync |
| `backend/api/*.py` | REST API blueprints |

---

## Azure OpenAI Configuration

```python
AZURE_OPENAI_ENDPOINT = "https://rishi-mihfdoty-eastus2.cognitiveservices.azure.com"
AZURE_API_VERSION = "2024-12-01-preview"
AZURE_CHAT_DEPLOYMENT = "gpt-5-chat"
AZURE_EMBEDDING_DEPLOYMENT = "text-embedding-3-large"
AZURE_WHISPER_DEPLOYMENT = "whisper"
```

### Embedding Dimensions Fix
Use `dimensions=1536` to match existing index:
```python
response = self.client.embeddings.create(
    model="text-embedding-3-large",
    input=query,
    dimensions=1536
)
```

---

## Session History

### December 5, 2024 - B2B SaaS Implementation
Built complete enterprise features:
- ✅ Phase 1: Database + JWT Authentication
- ✅ Phase 2: Gmail/Slack/Box Integrations
- ✅ Phase 3: Document Classification Flow
- ✅ Phase 4: Knowledge Gaps + Whisper + Index Rebuild
- ✅ Phase 5: Video Generation System

### December 2024 - Initial Setup
- Migrated from OpenAI to Azure OpenAI
- Fixed embedding dimension mismatch
- Fixed React hook ordering
- Pushed to GitHub

---

## GitHub Repository

https://github.com/rishitjain2205/2nd-brain

---

*Last updated: December 5, 2024*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Badrivinayakmishra)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Badrivinayakmishra)
<!-- tomevault:4.0:gemini_md:2026-04-08 -->
