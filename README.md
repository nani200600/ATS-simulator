# ATS Resume Checker — Production-Ready

A full-stack ATS resume analysis tool powered by Claude AI. Semantic NLP matching, ATS compatibility scanning, and actionable feedback — all in one.

---

## Architecture

```
ats-checker/
├── backend/          # Python FastAPI — Railway/Render
│   ├── app/
│   │   ├── main.py            # FastAPI app, middleware, CORS
│   │   ├── api/routes.py      # /analyze/text, /analyze/file, /health
│   │   ├── core/
│   │   │   ├── config.py      # Pydantic settings (env vars)
│   │   │   └── logging.py     # Structured logging (structlog)
│   │   ├── models/schemas.py  # All Pydantic request/response models
│   │   └── services/
│   │       ├── parser.py      # PDF (pdfminer + PyPDF2), DOCX, TXT extraction
│   │       ├── resume_analyzer.py  # Section detection, action verbs, NLP pre-analysis
│   │       └── ai_engine.py   # Claude API — semantic analysis core
│   ├── tests/test_api.py
│   ├── requirements.txt
│   ├── railway.toml
│   └── Procfile
└── frontend/         # React + TypeScript + Tailwind — Vercel
    ├── src/
    │   ├── App.tsx
    │   ├── types/             # TypeScript interfaces
    │   ├── hooks/useAnalysis.ts
    │   ├── services/api.ts    # Axios API layer
    │   └── components/
    │       ├── ScoreRing.tsx
    │       ├── SubScoreBars.tsx
    │       ├── KeywordPanel.tsx
    │       ├── CheckList.tsx
    │       ├── FileDropzone.tsx
    │       └── ResultsPanel.tsx
    └── vercel.json
```

---

## Scoring Engine

| Sub-Score | Weight | What it measures |
|---|---|---|
| Keyword Match | 35% | Exact + semantic keyword overlap |
| Semantic Similarity | 20% | Conceptual alignment via Claude NLP |
| ATS Compatibility | 20% | Formatting, sections, parsing safety |
| Content Quality | 15% | Action verbs, quantification, clarity |
| Formatting | 10% | Structure, length, standard headings |

Composite = weighted sum, 0–100. Score ≥ 80 = Excellent.

---

## Local Development

### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Download spaCy model
python -m spacy download en_core_web_sm

# Set env vars
cp .env.example .env
# Edit .env and set ANTHROPIC_API_KEY

uvicorn app.main:app --reload
# → http://localhost:8000
# → http://localhost:8000/docs  (Swagger UI)
```

### Frontend

```bash
cd frontend
npm install

# Set API URL
echo "VITE_API_URL=http://localhost:8000" > .env.local

npm run dev
# → http://localhost:5173
```

### Run Tests

```bash
cd backend
pytest tests/ -v
```

---

## Deployment

### Backend → Railway

1. Create a new Railway project
2. Connect your GitHub repo, set root directory to `backend/`
3. Add environment variables:
   ```
   ANTHROPIC_API_KEY=sk-ant-...
   ENVIRONMENT=production
   CORS_ORIGINS=["https://your-frontend.vercel.app"]
   ```
4. Railway auto-detects `Procfile` — deploy runs automatically
5. Note your Railway URL: `https://ats-checker-backend.railway.app`

### Frontend → Vercel

1. Import repo to Vercel, set root directory to `frontend/`
2. Add environment variable:
   ```
   VITE_API_URL=https://ats-checker-backend.railway.app
   ```
3. Deploy — Vercel auto-builds with Vite

---

## API Reference

### `POST /api/v1/analyze/text`
```json
{
  "resume_text": "John Doe...",
  "job_description": "We are looking for..."
}
```

### `POST /api/v1/analyze/file`
```
multipart/form-data:
  resume_file: <File: PDF/DOCX/TXT>
  job_description: <string>
```

### `GET /api/v1/health`
Returns service status and version.

---

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `ANTHROPIC_API_KEY` | ✅ | — | Your Anthropic API key |
| `ENVIRONMENT` | — | `development` | `production` disables docs |
| `CORS_ORIGINS` | — | `["http://localhost:5173"]` | Allowed frontend origins |
| `MAX_FILE_SIZE_MB` | — | `10` | Max upload size |
| `LOG_LEVEL` | — | `INFO` | Logging verbosity |

---

## Production Checklist

- [ ] `ANTHROPIC_API_KEY` set in Railway
- [ ] `CORS_ORIGINS` updated to your Vercel domain
- [ ] `ENVIRONMENT=production` (disables Swagger docs)
- [ ] `VITE_API_URL` set in Vercel to Railway URL
- [ ] Test `/api/v1/health` endpoint after deploy
- [ ] Test file upload with a real PDF resume
