# Thumbnail AI Analyzer (Starter)

This is a starter architecture for an AI SaaS platform that analyzes YouTube thumbnails and predicts their performance (CTR) using computer vision and psychology-inspired heuristics.

## High-Level Architecture

- **Backend**: FastAPI (`app/`)
  - `api/` – HTTP routes
  - `core/` – config and shared utilities
  - `services/` – image processing, feature extraction, scoring, suggestions
  - `db/` – database models and session
  - `schemas/` – Pydantic models for input/output
- **Image Processing**: OpenCV + Pillow
- **AI Models**: abstraction layer ready for Replicate / HuggingFace / other providers
- **Database**: PostgreSQL (via SQLAlchemy)

## Quickstart (Backend)

1. **Create and activate virtualenv** (recommended).
2. **Install dependencies**:

```bash
pip install -r requirements.txt
```

3. **Set environment variables** (for example, using `.env` or shell env):

- `APP_ENV=dev`
- `DATABASE_URL=postgresql+psycopg2://user:password@localhost:5432/thumbnail_ai`

4. **Run the API server**:

```bash
uvicorn app.main:app --reload
```

Then open the interactive docs at `http://localhost:8000/docs`.

## Core Endpoint

- `POST /analyze-thumbnail`
  - **Input**: image file (multipart/form-data, field name `file`)
  - **Output**: JSON containing:
    - extracted features
    - scoring breakdown
    - overall score
    - improvement suggestions

This starter implements heuristic logic that you can later replace with real AI model calls.

## Production configuration

For production, set the following in the environment (e.g. `.env` or deployment config):

| Variable | Description |
|----------|-------------|
| `APP_ENV=production` | Enables JWT secret validation and production behavior. |
| `JWT_SECRET_KEY` | **Required in production.** Must be at least 32 characters; use a strong random secret. |
| `CORS_ORIGINS` | Comma-separated allowed origins (e.g. `https://app.example.com,https://www.example.com`). Omit or set to `*` for development. |
| `DATABASE_URL` | PostgreSQL connection string. |
| `MAX_UPLOAD_BYTES` | Optional; default `5242880` (5MB). |
| `MAX_IMAGE_WIDTH` | Optional; default `1920`. |
| `MAX_IMAGE_HEIGHT` | Optional; default `1080`. |
| `REDIS_URL` | Optional; Redis URL for daily usage cache (e.g. `redis://localhost:6379/0`). If unset, usage limits use DB COUNT. |

- Uploaded filenames are sanitized (path traversal and unsafe characters removed).
- Image uploads are limited to 5MB and resized to fit within 1920×1080 before processing.
- `/api/thumbnails/recent` requires authentication and returns only the current user’s analyses.
- Unhandled errors return a generic message; details are logged server-side only.

## Database migrations (Alembic)

Schema changes are managed with [Alembic](https://alembic.sqlalchemy.org/).

- **First time or fresh DB:** from project root run  
  `alembic upgrade head`  
  (uses `DATABASE_URL` from env / `.env`).
- **Create a new migration** after editing `app/db/models.py`:  
  `alembic revision --autogenerate -m "description"`  
  then review the new file in `alembic/versions/` and run `alembic upgrade head`.
- If the DB already has the tables (e.g. from an older `init_db()`), align history without re-running DDL:  
  `alembic stamp 001`

See `alembic/README.md` for full commands and workflow.

## Frontend (Next.js)

- **Landing** (`/`): Public value proposition, Sign up / Log in. Authenticated users redirect to `/analyze`.
- **Analyze** (`/analyze`): Upload thumbnail (drag & drop), optional heatmap; 429 handling when daily limit is reached.
- **Results** (`/results`): Overall score, predicted CTR, radar breakdown, suggestions, heatmap. Supports `?id=<analysis_id>` to load from history.
- **History** (`/history`): List past analyses; View opens full result (from `GET /api/thumbnails/analyses/:id`).

## AI & CTR

- **Emotion:** Replicate facial-expression-recognition when `REPLICATE_API_TOKEN` is set; otherwise face-count heuristic. See `app/services/ai_providers/emotion.py`.
- **CTR:** XGBoost (or RandomForest fallback) trained on `thumbnail_analyses.ctr`. Persist with `CTR_MODEL_PATH`; optional `CTR_MODEL_VERSION`. See `docs/CTR_MODEL_TRAINING.md`.
- **Suggestions:** Rule-based plus optional tip from CTR feature importance. See `docs/SUGGESTIONS_RULES.md`.

## Tests & docs

- **Backend tests:** `pytest tests -v` (after `pip install -r requirements.txt`). Uses SQLite in-memory when `DATABASE_URL` is not set.
- **Docs:** `docs/API.md`, `docs/FEATURE_VECTOR_SCHEMA.md`, `docs/CTR_MODEL_TRAINING.md`, `docs/SUGGESTIONS_RULES.md`.

## Why this product?

Most creators struggle with low CTR due to weak thumbnails.

This tool helps by:
- Predicting CTR before publishing
- Analyzing visual psychology (faces, colors, text)
- Providing actionable improvements
- Learning over time using real data

Goal: Turn thumbnails into a data-driven process.

## AI Pipeline

1. Image Upload
2. Feature Extraction:
   - Faces & emotions
   - Text (OCR)
   - Colors & contrast
   - Saliency (attention focus)
3. Feature Vector Creation
4. CTR Prediction (XGBoost - future)
5. Suggestions (LLM + heuristics)

Future:
- Fully trained ML model based on collected dataset

## Example Response

```json
{
  "thumbnail_score": 78,
  "predicted_ctr": 6.2,
  "features": {
    "face_count": 1,
    "text_count": 3,
    "colorfulness": 0.65
  },
  "issues": [
    "Text too small",
    "Low contrast"
  ],
  "suggestions": [
    "Increase text size",
    "Use brighter colors"
  ]
}


---

## 4️⃣ 🗺️ Roadmap (مهم جدًا لك ومع Cursor)

```md
## Roadmap

### Phase 1 (Current)
- Heuristic scoring
- Feature extraction
- Dataset collection

### Phase 2
- Train CTR prediction model (XGBoost)
- Improve feature engineering

### Phase 3
- Replace heuristics with AI models
- Advanced visual analysis (saliency, emotion)

### Phase 4
- Full AI-powered optimization engine

## AI Usage

- Computer Vision: OpenCV, future deep models
- Emotion Detection: Replicate / HuggingFace
- OCR: Text extraction from thumbnails
- ML Model: XGBoost for CTR prediction
- LLM: Suggestions and improvements

## For AI Agents (Cursor)

- Follow .cursorrules strictly
- Read AGENT.md before making changes
- Keep AI modules modular
- Do not break existing endpoints