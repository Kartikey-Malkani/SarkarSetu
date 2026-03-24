# Sarkar Setu - Quick Start Guide

Get Sarkar Setu running locally in 5 minutes!

## Prerequisites
- Python 3.10+
- Node.js 18+
- Git

## Backend Setup (FastAPI)

```bash
# Navigate to backend directory
cd sarkar_setu

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Set environment variables (optional, defaults work fine)
# On Windows:
set SIMULATE_PORTAL=true
set RUN_EMBEDDING_PIPELINE=false

# On macOS/Linux:
export SIMULATE_PORTAL=true
export RUN_EMBEDDING_PIPELINE=false

# Start development server
python -m uvicorn src.api_server:app --reload

# The API will be available at http://localhost:8000
# Interactive docs: http://localhost:8000/docs
```

## Frontend Setup (Next.js)

In a new terminal:

```bash
# Navigate to frontend directory
cd sarkar_setu/frontend

# Install dependencies
npm install

# Set environment variable
# On Windows:
set NEXT_PUBLIC_API_URL=http://localhost:8000

# On macOS/Linux:
export NEXT_PUBLIC_API_URL=http://localhost:8000

# Start development server
npm run dev

# The frontend will be available at http://localhost:3000
```

## Testing the Integration

1. Open http://localhost:3000 in your browser
2. Try searching for schemes
3. Fill in your profile for personalized recommendations
4. Test the application submission (uses simulation mode by default)

## Troubleshooting

### Backend won't start
```bash
# Check Python version
python --version  # Should be 3.10+

# Reinstall dependencies
pip install -r requirements.txt --force-reinstall
```

### Frontend styles missing
```bash
# Reinstall npm dependencies
rm -rf node_modules
npm install

# Clear Next.js cache
rm -rf .next
npm run dev
```

### API connection fails
```bash
# Make sure NEXT_PUBLIC_API_URL is set correctly
# Check that backend is running on port 8000
curl http://localhost:8000/docs
```

## Production Deployment

See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) for Railway & Vercel deployment steps.

## Learn More

- [Architecture Overview](docs/ARCHITECTURE.md)
- [Features](docs/FEATURES.md)
- [API Documentation](docs/API_OVERVIEW.md)
- [Main README](README.md)

---

**Need help?** Open an issue on GitHub or email kartikeymalkani71@gmail.com
