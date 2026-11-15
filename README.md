[README.md](https://github.com/user-attachments/files/34868873/README.md)
# Universal AI Feedback Analyzer

Upload customer feedback CSVs, analyze them through multiple AI providers with automatic fallback, track progress via job-based polling, and visualize results with a premium dark-themed dashboard.

## Features

- **5 AI Providers**: OpenRouter, Gemini, Groq, Ollama (local), Anthropic (Claude)
- **Automatic Fallback**: If a provider fails, the system tries the next in chain
- **Rule-Based Fallback**: Always available — no API keys needed
- **Job-Based Progress**: Real-time batch progress polling (no WebSockets needed)
- **PII Masking**: Emails, phone numbers, order IDs masked before sending to AI
- **Smart Analysis**: Sentiment, sarcasm detection, spam flagging, rating-text conflicts
- **Report Generation**: Executive summaries with key findings and recommendations

## Quick Start

```bash
# 1. Copy environment variables
cp .env.example .env

# 2. Add your API keys to .env (optional — fallback works without keys)

# 3. Install dependencies
npm install
cd client && npm install
cd ../server && npm install
cd ..

# 4. Start dev servers
npm run dev
```

Frontend: http://localhost:5173
Backend: http://localhost:3001

## Privacy — PII Masking

Before sending any review text to external AI providers, the system masks personal information in a fixed order:

1. Order IDs (ORD123, Order #456, Customer ID 789)
2. Email addresses → `[EMAIL]`
3. Phone numbers (10-13 digits) → `[PHONE]`
4. Long numeric IDs (6+ digits) → `[NUMERIC_ID]`
5. Reviewer names (from structured CSV field) → `[NAME]`

Original text is stored internally and shown in the frontend. Only masked text is sent to AI.

## API Endpoints

| Method | Route | Purpose |
|--------|-------|---------|
| POST | /api/upload | Upload CSV file |
| POST | /api/upload/text | Upload raw CSV text |
| POST | /api/analyze/start | Start analysis job |
| GET | /api/analyze/status/:jobId | Job progress |
| GET | /api/analyze/result/:jobId | Final results |
| DELETE | /api/analyze/:jobId | Cleanup job |
| GET | /api/providers/status | Provider availability |
| POST | /api/report/generate | Generate report |
| GET | /api/sample/datasets | List samples |
| GET | /api/sample/dataset/:name | Get sample data |

## Known Limitations

- **In-memory job storage**: Jobs do not survive server restarts and are not shared across Node processes. For multi-process scaling, migrate to Redis or a database.
- **TTL eviction**: Completed/failed jobs are evicted after 1 hour.

## Architecture

```
feedback-analyzer/
├── client/               ← Vite + React 18
│   └── src/
│       ├── components/   ← 20 UI components
│       └── services/     ← API client
├── server/               ← Express
│   ├── services/         ← AI router, batch processor, job manager
│   │   └── providers/    ← 5 AI providers
│   ├── routes/           ← 5 route files
│   └── utils/            ← CSV parser, PII masker, JSON validator
└── .env                  ← Provider API keys
```
