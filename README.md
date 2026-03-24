# Sarkar Setu

> **Intelligent Government Scheme Application Portal with Automation & AI-Powered Recommendations**

An advanced web application that intelligently matches citizens with government schemes and automates application submissions across multiple portals. Built with modern tech stack (FastAPI, Next.js 16, React 19, Tailwind CSS 4) with production-ready deployment on Railway & Vercel.

![Status](https://img.shields.io/badge/status-production--ready-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Version](https://img.shields.io/badge/version-1.0.0-blue)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Screenshots](#screenshots)
- [Getting Started](#getting-started)
- [Deployment](#deployment)
- [API Overview](#api-overview)
- [Code Access Policy](#code-access-policy)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

---

## 🎯 Overview

**Sarkar Setu** ("Bridge to Government") bridges the gap between Indian citizens and government welfare schemes. It:

1. **Intelligently Recommends** schemes based on user demography and eligibility
2. **Automates Applications** with browser automation (Playwright) across multiple government portals
3. **Provides Real-Time Updates** on application status
4. **Ensures Multi-Language Support** for accessibility
5. **Prioritizes Privacy** with local computation and encrypted data handling

### Problem Statement
Citizens often don't know which government schemes they're eligible for, and manual application processes across multiple portals are time-consuming and error-prone.

### Solution
Sarkar Setu combines AI-powered recommendations with intelligent automation to make government benefits accessible and frictionless.

---

## ✨ Key Features

### 1. **Smart Scheme Recommendations**
- Semantic search using embeddings (Sentence-Transformers)
- Keyword-based fallback for lightweight deployments
- Real-time filtering by eligibility criteria
- Multi-language recommendations

### 2. **Automated Application Submission**
- Playwright-based browser automation
- Support for multiple government portals
- Error handling and retry mechanisms
- Application status tracking from DEMO_* format

### 3. **User Portal**
- Clean, responsive UI with Next.js 16 & React 19
- Real-time API integration
- Dark mode support with Framer Motion animations
- Accessibility-first design with Tailwind CSS 4

### 4. **Production-Ready Backend**
- FastAPI REST API with automatic OpenAPI docs
- Lightweight runtime mode (no heavy ML packages required)
- Graceful degradation to keyword search
- Configurable simulation mode for safe testing

### 5. **Security & Privacy**
- AES-256 encryption for sensitive data
- Environment-based configuration
- No personal data storage (stateless operation)
- Input validation and sanitization

### 6. **Observability**
- Vercel Web Analytics for frontend performance
- Request logging and error tracking
- Application submission audit trail

---

## 🛠️ Tech Stack

### Backend
- **Framework**: FastAPI 0.104+
- **Server**: Uvicorn
- **ML/NLP**: Sentence-Transformers, FAISS (optional for production)
- **Automation**: Playwright (optional, disabled by default)
- **Database**: Vector search (FAISS) + embeddings
- **Security**: Cryptography library for AES-256

### Frontend
- **Framework**: Next.js 16 (App Router)
- **UI Library**: React 19
- **Styling**: Tailwind CSS 4
- **Animations**: Framer Motion
- **Icons**: Lucide React
- **Analytics**: Vercel Web Analytics

### DevOps & Deployment
- **Backend Host**: Railway (Procfile + runtime.txt)
- **Frontend Host**: Vercel
- **Version Control**: Git/GitHub
- **Languages**: Python 3.10+ (backend), TypeScript (frontend)

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│              Frontend (Vercel)                      │
│  Next.js 16 + React 19 + Tailwind CSS 4           │
│  - User portal with scheme recommendations         │
│  - Application submission interface                │
│  - Real-time status updates via WebSocket           │
└────────────────────┬────────────────────────────────┘
                     │ HTTPS
                     ▼
┌─────────────────────────────────────────────────────┐
│              Backend API (Railway)                  │
│  FastAPI + Uvicorn                                 │
│  ├─ GET /schemes - List and search schemes        │
│  ├─ POST /recommend - Get personalized schemes    │
│  ├─ POST /apply - Automate application            │
│  └─ GET /docs - OpenAPI documentation             │
└────────────────────┬────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   ┌─────────┐  ┌──────────┐  ┌──────────────┐
   │ Schemes  │  │Embeddings │  │ Government   │
   │Database  │  │(FAISS VDB)│  │ Portals      │
   └─────────┘  └──────────┘  │ (Playwright) │
                               └──────────────┘
```

### Component Breakdown

- **Application Agent**: Orchestrates Playwright automation for form-filling
- **Query Engine**: Semantic search using embeddings (with lightweight fallback)
- **Scheme Database**: Curated list of government schemes with eligibility criteria
- **API Server**: FastAPI REST interface with async handlers

### Design Patterns

- **Lazy Loading**: Heavy ML modules loaded only when needed
- **Graceful Degradation**: Falls back to keyword search if embeddings unavailable
- **Simulation Mode**: Disabled automation by default (`SIMULATE_PORTAL=true`)
- **Environment Configuration**: All settings via environment variables

---

## 📸 Screenshots

Your product screenshots are located in the full repository. Below is what the application includes:

| Component | Description |
|-----------|-------------|
| **Dashboard** | Scheme search and recommendations interface |
| **Application Form** | Auto-filled application fields with validation |
| **Status Tracker** | Real-time track application status across portals |
| **Multi-Language UI** | Language selector for accessibility |

*See full screenshots and demo videos in the project repository*

---

## 🚀 Getting Started

### Prerequisites
- Python 3.10+ (backend)
- Node.js 18+ (frontend)
- Git
- Optional: Docker

### Local Development Setup

#### Backend
```bash
cd sarkar_setu
python -m venv venv
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

pip install -r requirements.txt
export SIMULATE_PORTAL=false  # Enable automation locally
python -m uvicorn src.api_server:app --reload
# API available at http://localhost:8000
```

#### Frontend
```bash
cd sarkar_setu/frontend
npm install
npm run dev
# Frontend available at http://localhost:3000
```

### Environment Variables

**Backend** (`.env`):
```
SIMULATE_PORTAL=true          # Default: simulation mode (disables automation)
RUN_EMBEDDING_PIPELINE=false  # Default: use lightweight keyword search
API_PORT=8000
```

**Frontend** (`.env.local`):
```
NEXT_PUBLIC_API_URL=http://localhost:8000  # Backend API endpoint
```

### Testing
```bash
# Backend
pytest sarkar_setu/tests/

# Frontend
npm run test
```

---

## 📦 Deployment

### Production Deployment

#### Railway (Backend)
- **Procfile**: `web: uvicorn src.api_server:app --host 0.0.0.0 --port $PORT`
- **runtime.txt**: `python-3.10`
- **Environment Variables**: Set `SIMULATE_PORTAL=true` and `RUN_EMBEDDING_PIPELINE=false` for lightweight production mode

**Full instructions**: See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md)

#### Vercel (Frontend)
- **Root Directory**: `sarkar_setu/frontend`
- **Build Command**: `npm run build`
- **Environment Variables**: Set `NEXT_PUBLIC_API_URL=<Railway_Backend_URL>`

**Lightweight Mode Benefits**:
- ⚡ Fast startup (< 2 minutes on Railway)
- 💾 Small container footprint (< 1 GB)
- 🔄 Graceful fallback to keyword search
- ✅ All features operational without ML packages

---

## 🔌 API Overview

### Base URL
```
https://api.sarkarsetu.example.com
```

### Key Endpoints

#### 1. Get Schemes
```http
GET /schemes
Content-Type: application/json

Response: { schemes: [...], total: 150 }
```

#### 2. Recommend Schemes
```http
POST /recommend
Content-Type: application/json

{
  "user_profile": {
    "age": 28,
    "income": 50000,
    "location": "Mumbai",
    "occupation": "Engineer"
  }
}

Response: { recommendations: [...], scores: [...] }
```

#### 3. Submit Application
```http
POST /apply
Content-Type: application/json

{
  "user_id": "USR123",
  "scheme_id": "SCHEME_001",
  "simulate_portal": false,
  "form_data": {...}
}

Response: { application_id: "DEMO_ABC123", status: "submitted" }
```

#### 4. API Documentation
```
GET /docs       # Interactive Swagger UI
GET /openapi.json  # OpenAPI schema
```

**Full API Reference**: See [docs/API_OVERVIEW.md](docs/API_OVERVIEW.md)

---

## 🔒 Code Access Policy

### This Repository (Documentation Showcase)
- ✅ **README & Documentation**: Freely accessible
- ✅ **Architecture Diagrams**: Freely accessible
- ✅ **API Specifications**: Freely accessible
- ✅ **Deployment Guides**: Freely accessible
- ✅ **Screenshots & Demo**: Freely accessible

### Source Code Access
- 🔐 **Full Source Code**: Confidential (proprietary)
- 📧 **Request Access**: Contact maintainer for evaluation

### Licensing
- This documentation and architecture is provided as-is for information purposes
- For code access or licensing inquiries, see [Contact](#contact)

### Open Source Components
This project builds on excellent open-source libraries:
- **FastAPI** (BSD 3-Clause)
- **Next.js** (MIT)
- **Tailwind CSS** (MIT)
- **Sentence-Transformers** (Apache 2.0)

See [LICENSE](LICENSE) and individual package licenses for details.

---

## 🤝 Contributing

We welcome contributions to documentation, architecture improvements, and feature ideas!

### How to Contribute
1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/my-contribution`)
3. **Commit** changes (`git commit -m 'docs: add feature explanation'`)
4. **Push** to branch (`git push origin feature/my-contribution`)
5. **Open** a Pull Request

### Guidelines
- See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines
- Follow the [Code of Conduct](CODE_OF_CONDUCT.md)
- Update documentation for any new features or changes
- Ensure all tests pass locally before submitting PR

### Development Focus Areas
- 🎨 UI/UX improvements for accessibility
- 🌍 Multi-language support expansion
- 📊 Analytics and reporting features
- ⚡ Performance optimization
- 🔒 Security enhancements

---

## 📄 License

This project is licensed under the **MIT License** - see [LICENSE](LICENSE) file for details.

### Summary
- ✅ You can use this code for commercial purposes
- ✅ You can modify and distribute
- ✅ You must include the original license and copyright notice
- ❌ The software is provided as-is without warranty

---

## 👨‍💻 Author & Contact

**Kartikey Malkani**
- 📧 Email: [kartikeymalkani71@gmail.com](mailto:kartikeymalkani71@gmail.com)
- 🔗 GitHub: [@Kartikey-Malkani](https://github.com/Kartikey-Malkani)
- 💼 LinkedIn: [Kartikey Malkani](https://linkedin.com/in/kartikey-malkani) *(if you have one)*

### For Inquiries
- **Feature Requests**: Open an issue on GitHub
- **Bug Reports**: Create a detailed issue with reproduction steps
- **Code Access/Partnership**: Email the maintainer
- **General Questions**: Check [Discussions](../../discussions) or create a Q&A issue

---

## 📚 Documentation

- [Architecture Deep Dive](docs/ARCHITECTURE.md)
- [Features Overview](docs/FEATURES.md)
- [Deployment Guide](docs/DEPLOYMENT.md)
- [API Reference](docs/API_OVERVIEW.md)
- [Contributing Guidelines](CONTRIBUTING.md)
- [Code of Conduct](CODE_OF_CONDUCT.md)

---

## 🎓 Learning Resources

### For Researchers & Students
- FastAPI best practices: https://fastapi.tiangolo.com
- Next.js 16 guide: https://nextjs.org
- Semantic search with embeddings: https://www.sbert.net
- Web automation with Playwright: https://playwright.dev

---

## ⭐ Show Your Support

If you find this project interesting, please consider:
- ⭐ Starring this repository
- 🔗 Sharing with your network
- 💬 Providing feedback and suggestions
- 🤝 Contributing to improvements

---

## 📋 Roadmap

- [ ] Mobile app (React Native)
- [ ] Additional government portals support
- [ ] Real-time application status WebSocket updates
- [ ] Video tutorials and documentation
- [ ] Community portal for scheme submissions
- [ ] Machine learning model fine-tuning tools

---

**Last Updated**: March 2026  
**Status**: Production Ready ✅

---

*Made with ❤️ for Indian citizens and digital accessibility*
