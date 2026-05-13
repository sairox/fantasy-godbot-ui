# Fantasy GodBot — Frontend

React + TypeScript chat UI for the Fantasy GodBot draft assistant.

## Commands

```bash
npm install        # install dependencies
npm run dev        # dev server at http://localhost:5173
npm run build      # production build
npm run lint       # eslint
npm run test       # vitest
```

## Architecture

```
User → ChatWindow → useChat hook → src/api/client.ts → FastAPI backend
                 → PickEvaluator → usePickEval hook  ↗
```

All backend calls go through `src/api/client.ts` — never call fetch directly from components.

## Key files

- `src/api/client.ts` — typed fetch wrappers for all backend endpoints
- `src/components/ChatWindow.tsx` — message thread display
- `src/components/ChatInput.tsx` — input bar + send button
- `src/components/PickEvaluator.tsx` — pick grade UI (grade, analysis, verdict)
- `src/hooks/useChat.ts` — session state, message history, loading/error state
- `src/types.ts` — shared TypeScript types matching backend response models

## Environment Variables

```
VITE_API_URL=http://localhost:8000   # backend base URL
```

## Backend API Contract

```ts
// POST /chat
{ message: string; league_format: "redraft" | "dynasty"; session_id: string }
→ { response: string; sources: string[] }

// POST /evaluate-pick
{ player_name: string; pick_number: number; league_format: "redraft" | "dynasty" }
→ { grade: string; analysis: string; verdict: string; raw_response: string }

// GET /player/{name}
→ { player_name: string; document: string }

// GET /health
→ { status: string; players_indexed: number; timestamp: string }
```

## Critical Invariants

- `session_id` must be stable per browser session — use `crypto.randomUUID()` stored in `sessionStorage`, not regenerated on every message.
- `league_format` is user-selectable (redraft/dynasty), persisted in `localStorage`.
- Never put API keys in the frontend — the backend handles all Claude/OpenAI calls.
- Streaming (`POST /chat/stream`) is planned on the backend but not yet live — use `/chat` until the backend adds the SSE endpoint, then update `client.ts` to use `ReadableStream`.

## Backend Handoff

Backend lives in the `fantasy-godbot` repo. CORS is already configured for `localhost:5173`. When the backend ships `POST /chat/stream`, update `client.ts` to stream tokens using `fetch` + `ReadableStream` — no library needed.