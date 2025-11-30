# NexFlow AI

NexFlow AI is a Cloudflare Workers automation builder with an AI “Flow Coach” that gives UI-aware guidance (no code generation). Users design cron-driven workflows, run them on demand, and review execution logs—all in one page backed by Workers, Durable Objects, Workflows, and Workers AI (Llama 3.1 8B Instruct).

## Live Preview
- Deployed app: `https://nexflow.mohamadmsalman82.workers.dev/`
- Try it fast:
  1) Open the URL above.
  2) In **AI Flow Coach** (top), ask for a starter flow idea (e.g., “Ping my API every 5 minutes and log status”).
  3) In **Design a workflow**, set **Flow name**, **Cron schedule (UTC)**, and build steps with **+ Insert**.
  4) In **Your flows**, select the flow, toggle **On**, and click **Run now**.
  5) In **Flow detail/logs**, expand recent runs to inspect step outputs/errors.

## Highlights (Recruiter Snapshot)
- **Full stack on Cloudflare**: Workers runtime, Durable Objects for state, Workflows cron for scheduling, Workers AI for the coach.
- **UI-aware AI**: Prompt constrains the model to give point-and-click instructions tied to visible controls.
- **Editable flows**: Create, edit, enable/disable, delete, run manually, and view history with per-step logs.
- **Typed & validated**: Zod schemas for all requests/responses; TypeScript across worker + frontend.
- **Embedded assets/prompts**: Frontend bundle and prompts inlined into the Worker—no extra hosting.

## Quick Start (Local)
```bash
npm install
npm run build           # bundles frontend & embeds assets/prompts
npm run dev             # wrangler dev (Worker + DO + Workflows shim)
# open the localhost URL from wrangler output
```
Other useful scripts:
- `npm run typecheck` — TS project-wide check
- `npm run cron:test` — cron parsing simulation
- `npm run build:frontend` — frontend-only build + embed
- `npm run deploy` — wrangler deploy

## Core Features
- **Flow builder**: Name, cron schedule, enabled toggle, and ordered steps:
  - `fetch` (method/url/headers/body/timeout)
  - `delay` (ms/s/m/h)
  - `condition` (input/operator/value) with short-circuit
  - `logic` (AND/OR multi-conditions) with short-circuit
  - `log` (message + include keys from prior steps)
  - `notify` (slack/discord/teams/webhook payloads)
- **Flow management**: Create, edit existing flows (prefill & PUT), enable/disable, delete, manual run.
- **Run history**: Status pills, per-step outputs/errors, console logs; capped history per flow.
- **AI Flow Coach**: UI-aware chat, seeded per-flow context, uses Workers AI `@cf/meta/llama-3.1-8b-instruct`.
- **Cron execution**: Cloudflare Workflows cron (`* * * * *`) plus in-Worker dev scheduler; `cron-parser` to decide “due” runs.

## Architecture (Code Map)
- **Worker entry**: `src/server.ts`  
  Serves SPA assets, `/api/flows`, `/api/chat`, `/api/flows/:id/run`, `/api/flows/:id/logs`, and starts a dev scheduler.
- **Durable Object**: `src/durable/NexFlowManager.ts`  
  Stores flows + run history; CRUD, enable toggle, manual runs; migration for legacy state.
- **Workflow cron engine**: `src/workflows/NexFlowEngine.ts`  
  Polls flows every minute, runs due ones, persists run records.
- **Flow execution**: `src/lib/stepExecutor.ts`  
  Implements fetch/condition/logic/delay/log/notify with short-circuit on condition/logic false; logs per step.
- **AI coach**: `src/lib/flowCoach.ts`, prompt source `src/prompts/flow-coach.txt`, compiled `src/generated/prompts.ts`.
- **Frontend**: `frontend/` (React + esbuild)  
  - `CreateFlow` — build/edit flows, live JSON preview  
  - `FlowList` — select/refresh/run/delete/toggle  
  - `FlowDetail` — step definitions + run history + “Edit flow”  
  - `FlowCoach` — chat UI with formatted output  
  - Styles: `frontend/styles.css`
- **Assets embedding**: `scripts/embed-assets.js` → `src/generated/assets.ts` (index/app.js/styles inlined).
- **Schemas**: `src/schemas/*` (FlowConfig, RunRecord, Chat, etc.) for runtime validation.

## API Surface (served by Worker)
| Method | Route | Description |
| --- | --- | --- |
| POST | `/api/chat` | UI-aware coach reply `{ reply }` |
| POST | `/api/flows` | Create flow |
| GET  | `/api/flows` | List flows |
| GET  | `/api/flows/:id` | Fetch flow |
| PUT  | `/api/flows/:id` | Update flow |
| DELETE | `/api/flows/:id` | Delete flow and logs |
| PATCH | `/api/flows/:id/enabled` | Enable/disable |
| POST | `/api/flows/:id/run` | Manual run |
| GET  | `/api/flows/:id/logs` | Run history |

## Cloudflare Bindings (wrangler.jsonc)
- `AI` — Workers AI (`@cf/meta/llama-3.1-8b-instruct`)
- `NEXFLOW_MANAGER` — Durable Object namespace
- `NEXFLOW_ENGINE` — Workflow binding for cron execution

## Development Notes
- Frontend is bundled to `frontend/dist/` then embedded; run `npm run build` before deploy.
- Generated files live under `src/generated/` (assets/prompt). Edit sources, not generated outputs.
- Avoid committing `node_modules/`, `frontend/dist/`, `.wrangler/` (ignored).

## Manual Smoke Test
1) `npm run dev`
2) Create or edit a flow (set name, cron, steps).
3) Ask the AI Flow Coach how to adjust fields; responses should reference on-screen controls.
4) Click **Run now**; check Flow detail/logs for per-step outputs/errors.
5) Leave dev running ~1 minute to see cron-triggered runs in logs.

---
For questions or deeper review, start with `src/server.ts` (API), `src/durable/NexFlowManager.ts` (state), `src/lib/stepExecutor.ts` (execution), and `frontend/app.tsx` (composition). Recruiters: the live link above shows the full experience without setup.
