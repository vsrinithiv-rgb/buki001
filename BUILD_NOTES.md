# Step 3 Build Notes — Bring Sage into Buki

**Date:** Nov 25, 2026  
**Stack:** MongoDB, Mongoose, Express.js, Node.js, React (Vite)

---

## 1. Pre-Code Test List Status

> **Environment Note:** During generation in this workspace, terminal execution (`run_command`) was unavailable because `powershell` could not be resolved in `%PATH%`. All automated test cases were implemented in `backend/tests/api.test.js` using Node's built-in `node:test` runner, but **have not yet been executed**. Run `cd backend && npm install && npm test` locally to execute and verify all checks.

- [ ] **Valid Query Check** (`tests/api.test.js` — Test 1): Verifies `POST /api/query` with a valid `BUKI_API_TOKEN` returns `200 OK`, contextual markdown answer, citations, latency metadata, and forwards `Authorization: Bearer <SAGE_API_KEY>` to Sage.
- [ ] **Malformed/Empty Query Validation** (`tests/api.test.js` — Test 2): Verifies empty string, whitespace-only, 1-char string, non-string (`12345`), missing `query`, and malformed raw JSON all return `400 Bad Request` with `VALIDATION_ERROR`.
- [ ] **Authentication Check** (`tests/api.test.js` — Test 3): Verifies missing `Authorization` header, non-Bearer scheme, and invalid Bearer token return `401 Unauthorized` with `UNAUTHORIZED`. Frontend `AuthGate` blocks unauthenticated views and redirects back to sign-in on `401`.
- [ ] **Latency & Concurrency Check** (`tests/api.test.js` — Tests 4a & 4b): Verifies `AbortController` aborts stalled Sage requests at the configured timeout (`3000ms` default) returning `504 SAGE_TIMEOUT`, and confirms 10 concurrent valid queries resolve cleanly.

---

## 2. MVP Cut Decisions

1. **No Multi-Turn Conversation Caching:** Kept queries stateless single-turn (`POST /api/query`) instead of implementing Redis/MongoDB multi-turn thread memory, keeping latency predictable and backend complexity minimal.
2. **No Token-by-Token Streaming (SSE):** Used standard JSON request/response with frontend loading skeleton states to align with the 3-second SLA target and simpler error/timeout recovery.
3. **Lightweight Bearer Token Gate vs. Full SSO:** Used `BUKI_API_TOKEN` constant-time Bearer validation (`crypto.timingSafeEqual`) in `backend/middleware/auth.js` with `req.user` attachment, making it easy to swap in JWT/OAuth verification later without changing route handlers.
4. **Graceful Optional MongoDB Logging:** `backend/config/db.js` and `backend/routes/query.js` persist `QueryLog` records when MongoDB is connected, but degrade gracefully without blocking `/api/query` if MongoDB is offline during quick local evaluation.

---

## 3. AI Assistance & Manual Verification Checklist

- **AI Assistance Used:** Generated full MERN boilerplate, modular `SageConnector` with `AbortController` timeout and mock mode, input/auth middlewares, React UI components, and `node:test` suite.
- **Manual Verification To Run Locally:**
  1. `cd backend && cp .env.example .env && npm install && npm test`
  2. `npm run dev` (verify `GET http://localhost:5000/api/health` returns `200 OK`)
  3. `cd frontend && npm install && npm run dev`
  4. Test sign-in with invalid token (expect inline error), valid token (`buki-dev-internal-token-change-me`), empty submit (expect validation banner), valid prompt (expect markdown response + latency badge), and `Ctrl+K` shortcut.
