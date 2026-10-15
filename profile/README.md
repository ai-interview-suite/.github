# AI Interview Suite

AI-powered mock technical interviews. Upload a resume and a job description, answer five
generated questions, and get a scored report with per-question feedback and a hiring
recommendation.

## Live

| | URL |
| --- | --- |
| App | https://aiinterviewsuite.vercel.app |
| API | https://ai-interview-suite-api.onrender.com |
| API docs | https://ai-interview-suite-api.onrender.com/docs |

> The API runs on Render's free plan, so it sleeps when idle — the first request after a
> quiet period can take up to a minute.

## Repositories

| Repo | Stack | What it is |
| --- | --- | --- |
| [`front`](https://github.com/ai-interview-suite/front) | Next.js 16, React 19, Tailwind CSS 4 | **InterviewPrep** — the web front end: sign-up, resume upload, the question flow and the results report |
| [`api`](https://github.com/ai-interview-suite/api) | FastAPI, Python 3.12, PostgreSQL | **AI Interview Service** — auth, interview storage, question generation and answer evaluation |

## How it works

1. **Start** — the resume PDF is parsed to text and sent with the job description to the
   model, which returns exactly five questions, each tagged with a category (technical,
   behavioral, experience, problem solving, system design), a difficulty and its expected
   topics.
2. **Answer** — answers are submitted one at a time against the interview id.
3. **Submit** — once every question is answered, each answer is scored and the results are
   aggregated into an overall score and a recommendation.

Each answer is rated 0–10 on four axes, combined into a 0–100 score:

| Axis | Weight |
| --- | --- |
| Technical accuracy | 40% |
| Relevance | 25% |
| Completeness | 20% |
| Communication | 15% |

The overall score is the mean of the per-answer scores, which maps to a recommendation of
`excellent`, `good`, `borderline` or `needs_improvement`.

## Architecture

```
front (Vercel)  ──HTTPS──>  api (Render)  ──>  PostgreSQL (Neon, ap-southeast-1)
                                          └──>  OpenRouter (question generation + evaluation)
```

Users register with email and password; the API issues a JWT, and interviews are stored in
PostgreSQL scoped to the user who created them.

## Running it locally

Start with the API — the front end needs it running.

```bash
# 1. API — requires Python 3.12 and a running PostgreSQL server
git clone https://github.com/ai-interview-suite/api
cd api
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env     # fill in DATABASE_URL, OPENROUTER_API_KEY, JWT_SECRET_KEY, ...
alembic upgrade head
uvicorn app.main:app --reload

# 2. Front end
git clone https://github.com/ai-interview-suite/front
cd front
npm install
echo "NEXT_PUBLIC_API_BASE_URL=http://localhost:8000" > .env.local
npm run dev
```

Open http://localhost:3000 — use `localhost`, not `127.0.0.1`, since that is the origin
listed in the API's `CORS_ORIGINS`.

Each repo's README has the full detail: every environment variable, the Neon and Render
setup, the endpoint reference and the test database.

## Deployment

| Part | Host | Notes |
| --- | --- | --- |
| Front end | Vercel | From the `front` repo; `NEXT_PUBLIC_API_BASE_URL` is baked in at build time |
| API | Render web service | From the `api` repo; auto-deploys on every commit to `main` |
| Database | Neon serverless PostgreSQL | AWS ap-southeast-1 (Singapore) |

Two things that bite: Alembic migrations are **not** run by Render, so apply them to Neon
from your machine before merging a revision; and every front-end origin — production,
preview or custom domain — has to be added to the API's `CORS_ORIGINS`.
