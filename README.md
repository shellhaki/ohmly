# Ohmly — Project Overview

## What it is
Ohmly is an AI tool that generates Wokwi-compatible `sketch.ino` and `diagram.json` files from natural language descriptions, so hardware/firmware engineers can go from an idea straight to a running simulation.

## Target users
Professional hardware/firmware engineers who currently hand-build Wokwi projects (wiring diagrams + firmware code) and want to skip the boilerplate setup.

## Core UX
1. User describes what they want to build (in plain language)
2. AI asks intelligent clarifying questions where needed (e.g. recommending a specific microcontroller)
3. AI generates a working `diagram.json` + `sketch.ino` in seconds, ready to paste into Wokwi and run

Positioned as "vibe designing" for hardware — the AI-assisted-coding experience, applied to circuit + firmware generation.

## Scope
- First version: MVP to demo within a few weeks
- Longer-term: possibly a proprietary circuit simulator/IDE, instead of relying solely on Wokwi

## Tech stack
- **Backend:** Bun + Hono + TypeScript
- **Frontend:** Next.js + TypeScript + Tailwind + shadcn/ui
- **Database:** Postgres
- **Sessions/cache:** Redis (cookie-based sessions, OTP storage)
- **AI model:** Gemini

## Auth model
Email + OTP, no passwords. Sessions stored in Redis, delivered via httpOnly cookie. See `ohmly-backend-plan.md` for full endpoint list.

## Brand direction
Soft-3D, pink/purple aesthetic with a robot mascot — playful and approachable, deliberately distinct from the founder's other product (SparkDB), which uses a minimal monochrome look. See `ohmly-design-context.md` for full design brief.

## Related docs
- `ohmly-backend-plan.md` — full API/endpoint plan
- `ohmly-design-context.md` — design brief for UI generation (Claude Code)