# Ohmly — Backend Plan

Stack: Bun + Hono + TypeScript, Postgres, Redis. AI model: Gemini.

## 1. Auth Service (Redis + cookie sessions + OTP)

Email/OTP-based auth — no passwords.

| Endpoint | Method | Purpose |
|---|---|---|
| `/auth/otp/request` | POST | Takes email, generates OTP, stores in Redis (TTL ~5 min), sends via email provider. Rate-limited to 1 request / 60s per email. |
| `/auth/otp/verify` | POST | Takes email + OTP, validates against Redis, creates session on success, deletes OTP key. |
| `/auth/session` | POST (internal) | Creates session: generates session ID, stores `session:{id} → {userId, createdAt}` in Redis (TTL 7–30 days), sets httpOnly/secure cookie. |
| `/auth/me` | GET | Reads session cookie, looks up Redis, returns current user. |
| `/auth/logout` | POST | Deletes Redis session key, clears cookie. |
| `/auth/session/refresh` | POST | Sliding expiration — extends Redis TTL on activity (optional). |

**Redis key design:**
- `otp:{email}` → code, TTL 5 min, rate-limited
- `session:{sessionId}` → JSON blob, TTL 7–30 days
- `user_sessions:{userId}` → set of session IDs, enables "log out everywhere"

## 2. Project Utilities

| Endpoint | Method | Purpose |
|---|---|---|
| `/projects` | POST | Create project (name, description) |
| `/projects` | GET | List user's projects |
| `/projects/:id` | GET | Get project detail incl. latest sketch.ino/diagram.json |
| `/projects/:id` | PATCH | Rename/update metadata |
| `/projects/:id` | DELETE | Delete project |
| `/projects/:id/versions` | GET | List generation version history |

## 3. AI Generation Core

| Endpoint | Method | Purpose |
|---|---|---|
| `/projects/:id/generate` | POST | Takes natural language prompt, calls Gemini, returns clarifying questions OR a draft diagram.json + sketch.ino |
| `/projects/:id/generate/clarify` | POST | User answers clarifying questions, continues the generation loop |
| `/projects/:id/generate/confirm` | POST | Finalizes and persists generated files as a new version |
| `/projects/:id/files` | GET | Fetch current sketch.ino + diagram.json |
| `/projects/:id/regenerate` | POST | Regenerate with tweaks/feedback on an existing version |

**Generation state:** in-progress generation context lives in Redis (`gen:{sessionId}` → conversation history + partial spec), committed to Postgres only once confirmed — keeps Postgres free of abandoned attempts.

## 4. Postgres Schema (core tables)

- `users` (id, email, created_at)
- `projects` (id, user_id, name, description, created_at)
- `project_versions` (id, project_id, sketch_ino, diagram_json, prompt_used, created_at)

Sessions and OTPs live only in Redis — no duplication in Postgres.