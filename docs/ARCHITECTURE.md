# Sarkar Setu - Architecture Deep Dive

## System Overview

Sarkar Setu is a distributed web application with a clear separation between frontend and backend, each optimized for their specific domain.

```
┌──────────────────────────────────┐
│     Client Layer (Vercel)        │
│  Next.js 16 + React 19           │
│  Tailwind CSS 4 + Framer Motion  │
└────────────┬─────────────────────┘
             │ HTTPS REST API
             ▼
┌──────────────────────────────────┐
│     API Layer (Railway)          │
│  FastAPI + Uvicorn               │
│  OpenAPI Documentation           │
└────────────┬─────────────────────┘
             │
    ┌────────┴────────┐
    ▼                 ▼
┌──────────────┐  ┌──────────────┐
│ Data Layer   │  │ External API │
│ (In-Memory   │  │ (Government  │
│  + Vector    │  │  Portals)    │
│  Database)   │  │ (Playwright) │
└──────────────┘  └──────────────┘
```

## Architecture Components

### 1. Frontend Architecture (Next.js 16)

#### App Structure
```
frontend/src/
├── app/
│   ├── layout.tsx           # Root layout with providers
│   ├── page.tsx             # Home page
│   └── [routes]/            # Dynamic routes
├── components/
│   ├── ui/                  # Reusable UI components
│   ├── forms/               # Form components with validation
│   ├── schemes/             # Scheme-related components
│   └── common/              # Common components (header, footer)
├── lib/
│   ├── api.ts               # API client (Axios/Fetch)
│   ├── types.ts             # TypeScript interfaces
│   └── utils.ts             # Utility functions
└── styles/
    └── globals.css          # Tailwind CSS imports
```

#### Key Design Patterns

**1. Server Components + Client Components**
- Use Server Components for:
  - Fetching data
  - Accessing backend secrets
  - Rendering static content
- Use Client Components for:
  - Interactive elements
  - Event handlers
  - State management

**2. API Integration Layer**
```typescript
// lib/api.ts
const BASE_URL = process.env.NEXT_PUBLIC_API_URL || "";

export async function fetchSchemes() {
  const response = await fetch(`${BASE_URL}/schemes`);
  return response.json();
}
```

**3. Component Composition**
- Small, focused components
- Props-based configuration
- Reusable across pages

#### Styling Stack
- **Tailwind CSS 4** with PostCSS plugin
- **Dark Mode**: Built-in Tailwind dark mode
- **Animations**: Framer Motion for micro-interactions
- **Responsive**: Mobile-first design approach

### 2. Backend Architecture (FastAPI)

#### Project Structure
```
sarkar_setu/
├── src/
│   ├── api_server.py          # FastAPI app initialization
│   ├── application_agent.py   # Playwright automation orchestration
│   ├── scheme_automations.py  # Scheme-specific automation rules
│   ├── sarkar_setu_engine.py  # Query engine with fallback
│   ├── embedding_pipeline.py  # Embedding generation (optional)
│   ├── query_engine.py        # Semantic search with FAISS
│   ├── lightweight_engine.py  # Keyword-based fallback
│   └── utils/
│       ├── config.py          # Configuration management
│       ├── crypto.py          # Encryption utilities
│       └── validators.py      # Data validation
├── schemes_data/
│   ├── schemes.json           # Scheme definitions
│   └── eligibility_rules.py   # Eligibility logic
└── requirements.txt
```

#### Core Modules

**1. Application Agent** (`application_agent.py`)
```
Responsibilities:
├── Detect scheme type from application
├── Route to correct automation handler
├── Execute form-filling with Playwright
├── Handle captchas and validations
└── Return completion status
```

**2. Query Engine** (`sarkar_setu_engine.py`)
```
Query Engine Hierarchy:
├── Try: Full QueryEngine (Sentence-Transformers + FAISS)
│   └─ Semantic similarity search, high accuracy
└── Fallback: LightweightQueryEngine (Keyword-based)
    └─ Keyword overlap scoring, no ML dependencies
```

**3. API Server** (`api_server.py`)
```
Endpoints:
├── GET /schemes
│   └─ List all schemes with metadata
├── GET /schemes/{id}
│   └─ Get scheme details
├── POST /recommend
│   └─ Get personalized recommendations
├── POST /apply
│   └─ Submit application with automation
├── GET /status/{application_id}
│   └─ Check application status
└── GET /docs
    └─ Interactive API documentation
```

#### Production Features

**1. Lightweight Runtime Mode**
```
Environment Variable: RUN_EMBEDDING_PIPELINE=false

Impact:
├── Skip Sentence-Transformers loading
├── Skip FAISS index building
├── Use keyword search for recommendations
├── Reduce startup time: 5s → 1s
└── Reduce memory usage: 2GB → 200MB
```

**2. Simulation Mode**
```
Environment Variable: SIMULATE_PORTAL=true

Impact:
├── Skip actual Playwright execution
├── Return demo submission IDs (DEMO_*)
├── Avoid accessing external government portals
├── Perfect for testing and demo deployments
└── Zero external dependencies needed
```

**3. Configuration Management**
```python
# Environment variables with defaults
SIMULATE_PORTAL = os.getenv("SIMULATE_PORTAL", "true") == "true"
RUN_EMBEDDING_PIPELINE = os.getenv("RUN_EMBEDDING_PIPELINE", "false") == "true"
API_PORT = int(os.getenv("API_PORT", "8000"))
```

#### Data Models

**User Profile**
```python
class UserProfile(BaseModel):
    age: int
    income: int
    state: str
    occupation: str
    education_level: str
    family_size: int
    caste: Optional[str] = None
    gender: str
```

**Scheme**
```python
class Scheme(BaseModel):
    id: str
    name: str
    ministry: str
    description: str
    eligibility_criteria: dict
    benefit_amount: float
    portal_url: str
```

**Application**
```python
class Application(BaseModel):
    id: str
    user_id: str
    scheme_id: str
    status: str  # submitted, processing, approved, rejected
    submission_date: datetime
    data: dict
```

### 3. Data Flow

#### Recommendation Flow
```
User Input (UserProfile)
    ↓
API /recommend endpoint
    ↓
Query Engine
├─ Main Path: Semantic Search
│  ├─ Convert to embeddings
│  ├─ Search FAISS index
│  └─ Return top-K results
└─ Fallback Path: Keyword Search
   ├─ Keyword overlap scoring
   ├─ Rank by relevance
   └─ Return top-K results
    ↓
Filter by eligibility rules
    ↓
Sort by recommendation score
    ↓
Return to frontend (JSON)
```

#### Application Submission Flow
```
User Selects Scheme
    ↓
API /apply endpoint with form data
    ↓
Check SIMULATE_PORTAL flag
├─ If true: Return DEMO_ABC123 (demo mode)
└─ If false: Execute automation
    ├─ Initialize Playwright browser
    ├─ Navigate to government portal
    ├─ Fill form fields automatically
    ├─ Handle validations/captchas
    ├─ Submit application
    └─ Return submission ID
    ↓
Store application metadata
    ↓
Return status to frontend
```

### 4. Integration Points

#### External Integrations

**1. Government Portals**
- Dynamic form detection
- Element locator strategy
- Multi-state portal variations
- Captcha handling (optional service)

**2. SMS/Email Notifications** (Optional)
- Application submission confirmation
- Status updates
- Eligibility alerts

### 5. Security Architecture

#### Data Protection

**1. Encryption**
```python
from cryptography.fernet import Fernet

cipher = Fernet(encryption_key)
encrypted_data = cipher.encrypt(sensitive_data.encode())
decrypted_data = cipher.decrypt(encrypted_data).decode()
```

**2. Input Validation**
- Pydantic models for schema validation
- Sanitization of form inputs
- XSS prevention

**3. CORS Security**
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://frontend.vercel.app"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

#### Privacy by Design
- No persistent storage of personal data (stateless)
- Data encrypted in transit (HTTPS only)
- Simulation mode for testing without data exposure
- No logging of sensitive information

### 6. Performance Optimization

#### Backend Optimizations

**1. Lazy Loading**
```python
# Don't import until needed
def ensure_pipeline_ready():
    if RUN_EMBEDDING_PIPELINE:
        from embedding_pipeline import EmbeddingPipeline
        return EmbeddingPipeline()
```

**2. Graceful Degradation**
```python
try:
    engine = QueryEngine()  # Full semantic search
except ImportError:
    engine = LightweightQueryEngine()  # Keyword search
```

**3. Async Handlers**
```python
@app.post("/recommend")
async def recommend(profile: UserProfile):
    # Non-blocking execution
    results = await query_engine.search(profile)
    return results
```

#### Frontend Optimizations

**1. Code Splitting**
- Route-based code splitting (Next.js automatic)
- Dynamic imports for heavy components

**2. Image Optimization**
- Next.js Image component with lazy loading
- Automatic format selection (WebP, etc.)

**3. Caching Strategy**
```typescript
// Cache scheme list for 1 hour
const schemes = await fetch(url, {
  next: { revalidate: 3600 }
});
```

### 7. Deployment Architecture

#### Railway Backend
```
Repository (GitHub)
    ↓
Railway Deployment
├─ Procfile: uvicorn startup command
├─ runtime.txt: Python 3.10
└─ Environment Variables
   ├─ SIMULATE_PORTAL=true
   ├─ RUN_EMBEDDING_PIPELINE=false
   └─ [Custom variables]
    ↓
Docker Container (auto-generated)
├─ Lightweight image (< 1 GB)
├─ Fast build time (< 2 min)
└─ Dynamic PORT binding
    ↓
Exposed at: https://railway-url/api
```

#### Vercel Frontend
```
GitHub Repository
    ↓
Vercel Deployment
├─ Root: /sarkar_setu/frontend
├─ Build: npm run build
└─ Environment Variables
   ├─ NEXT_PUBLIC_API_URL=railway-url
   └─ [Analytics tokens]
    ↓
Build Process
├─ TypeScript compilation
├─ PostCSS/Tailwind compilation
├─ Static generation
└─ Edge caching
    ↓
Deployed at: https://vercel-url
```

## Design Decisions & Rationale

| Decision | Rationale |
|----------|-----------|
| FastAPI over Django | Lightweight, async-native, auto OpenAPI docs |
| Next.js 16 over CRA | Server components, built-in optimization, ISR |
| Tailwind CSS 4 | Utility-first, no bloated CSS, great DX |
| FAISS for vectors | Fast approximate search, no external DB needed |
| Playwright for automation | Chromium-based, reliable, cross-platform |
| Simulation mode by default | Safe production deployment, demo-friendly |
| Keyword search fallback | Zero dependency degradation path, always functional |

## Scalability Considerations

### Current Limitations
- Embeddings stored in-memory (FAISS index)
- No horizontal scaling of backend
- Single government portal at a time

### Future Improvements
- Vector database (Pinecone, Weaviate) for distributed embeddings
- Kubernetes deployment for autoscaling
- Message queue (Celery) for long-running tasks
- Webhook system for portal updates
- Caching layer (Redis) for recommendation queries

## Monitoring & Observability

### Frontend (Vercel Analytics)
- Page views, interactions
- Core Web Vitals
- User sessions

### Backend (Application-level)
- API request latency
- Error rates
- Recommendation quality metrics
- Application submission success rate

## References

- [FastAPI Documentation](https://fastapi.tiangolo.com)
- [Next.js 16 Docs](https://nextjs.org)
- [Tailwind CSS Docs](https://tailwindcss.com)
- [Playwright Docs](https://playwright.dev)
- [FAISS Documentation](https://github.com/facebookresearch/faiss)
