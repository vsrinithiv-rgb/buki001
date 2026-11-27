# Bring Sage into Buki

A lightweight internal assistant tool integrating Sage into Buki's workflows using the MERN stack (MongoDB, Express.js, React, Node.js).

## Project Structure

- `backend/` — Express API server, Bearer auth middleware, input validation, Mongoose `QueryLog` model, modular `SageConnector` (with 3-second `AbortController` timeout), and Node built-in test suite (`tests/api.test.js`).
- `frontend/` — Vite + React minimalist internal assistant UI with session token gate, sticky composer (`Ctrl+K` / `Cmd+K`, `Enter` to submit), loading skeletons, inline error banners, and GitHub-flavored Markdown rendering.
- `BUILD_NOTES.md` — Step 3 execution log, cut decisions, and pre-code test checklist.

## Prerequisites

- **Node.js** `>= 18.0.0` (uses built-in `fetch`, `AbortController`, and `node --test`)
- **MongoDB** (optional for mock/stateless runs; when running locally, query logs are persisted to `QueryLog`)

## Quick Start

### 1. Backend Setup

```bash
cd backend
cp .env.example .env
npm install
npm test
npm run dev
```

By default, `backend/.env.example` sets `SAGE_MOCK_MODE=true` so you can verify the full request/response flow immediately without live Sage credentials. To connect a live Sage endpoint, update `SAGE_API_URL`, `SAGE_API_KEY`, and set `SAGE_MOCK_MODE=false`.

### 2. Frontend Setup

```bash
cd frontend
cp .env.example .env
npm install
npm run dev
```

Open `http://localhost:5173` and authenticate with your `BUKI_API_TOKEN` (default dev token in `backend/.env.example`: `buki-dev-internal-token-change-me`).

### 3. Single-Command Production Build & Start (Render / Unified Host)

From the repository root:

```bash
npm install && npm run build && npm start
```

Express automatically serves the built `frontend/dist` bundle on `/` and API routes on `/api/*`. See `render.yaml` for Render Web Service deployment settings.

## API Contract

### `GET /api/health`
Returns service status, database connectivity, and current Sage mode/timeout.

### `POST /api/query`
Requires `Authorization: Bearer <BUKI_API_TOKEN>`.

Request:
```json
{
  "query": "How do I complete Buki engineering onboarding?",
  "context": null
}
```

Response (`200 OK`):
```json
{
  "ok": true,
  "data": {
    "answer": "### Buki Internal Onboarding Checklist...",
    "citations": [{ "title": "Engineering Onboarding Runbook", "source": "Buki Wiki / Eng-Ops" }],
    "meta": {
      "requestId": "sage_...",
      "latencyMs": 46,
      "provider": "sage-mock",
      "timestamp": "2026-11-25T06:00:00.000Z"
    }
  }
}
```
