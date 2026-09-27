# MemoryGate

**AI Runtime Governance** — the layer that sits between autonomous AI agents and the
systems they act on, evaluating every request in real time: who the agent is, what it's
trying to access, why, which policies apply, and whether the action should be **allowed,
modified, escalated, or blocked**.

Framework-agnostic by design. Drop-in middleware ships for **LangGraph**, the **OpenAI
Agents SDK**, **CrewAI**, **Google ADK**, and **MCP** — the same policy engine governs an
agent's actions no matter which of those it's built on.

## What's in this repo

```
backend/      FastAPI + MongoDB (Motor). Multi-tenant auth, policy engine, decision API,
              webhooks, connectors, RBAC, compliance export, Dodo Payments billing.
frontend/     React 19 console — policy builder, live decision stream, risk heatmap,
              compliance center, public no-signup playground.
sdks/python/  `memorygate` — sync + async clients, typed `Decision` model, framework
              middleware for LangGraph / OpenAI Agents / CrewAI / Google ADK / MCP.
sdks/typescript/  `@memorygate/sdk` — ESM + CJS, typed exceptions, tsup build.
benchmarks/   Latency/throughput harness (p50 47ms single-caller, 159 req/s @ 16-way).
tests/        Backend pytest suite — auth, CORS/CSRF, policy engine, Dodo Payments, Breeth memory, smoke tests.
memory/       Product spec and build log (PRD.md).
```

## Core capabilities

- **Runtime decision engine** — ordered, priority-based policies matched on resource
  pattern, action, subject, and conditions (risk score, agent trust, purpose, etc.),
  returning an explainable evaluation trace.
- **Multi-tenant auth & RBAC** — JWT (httpOnly cookie + Bearer fallback), 4 roles
  (owner/admin/editor/viewer) enforced on every mutation.
- **Policy versioning & rollback** — every change is snapshotted; roll back to any
  prior version.
- **Compliance center** — SOC 2 Type II / ISO 27001:2022 / GDPR / HIPAA controls mapped
  from live runtime evidence, exportable as CSV.
- **Webhooks & connectors** — Slack / Teams / PagerDuty / custom delivery; PostgreSQL /
  MongoDB / SurrealDB / Redis / Pinecone / Qdrant / REST connectors.
- **Public playground** — no-signup, rate-limited live demo of the decision engine at
  `/playground`.

## Quick start

Full walkthrough (VS Code, both terminals, env setup) is in **[LOCAL_SETUP.md](./LOCAL_SETUP.md)**.
Short version:

```bash
# backend
cd backend
cp .env.example .env   # then edit: JWT_SECRET, MONGO_URL, ADMIN_EMAIL/PASSWORD
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python -m uvicorn server:app --reload --host 127.0.0.1 --port 8001

# frontend, in a second terminal
cd frontend
cp .env.example .env
yarn install
yarn start
```

App: `http://localhost:3000` · API docs: `http://localhost:8001/api/docs`

Payments (`/api/payments/*`) use the official `dodopayments` SDK directly — set
`DODO_PAYMENTS_API_KEY` and `DODO_PAYMENTS_WEBHOOK_KEY` in `backend/.env` to exercise those routes;
every other route works without them.

## SDKs

- **Python** — `pip install -e ./sdks/python`, see [sdks/python/README.md](./sdks/python/README.md).
- **TypeScript** — `cd sdks/typescript && yarn install`, see [sdks/typescript/README.md](./sdks/typescript/README.md).

## Testing

```bash
cd backend && python -m pytest tests/ -n 2 --dist loadscope
```

Requires a running backend at `REACT_APP_BACKEND_URL` (defaults to `http://localhost:8001`).
See [memory/PRD.md](./memory/PRD.md) for the full build log and what each test suite covers.

## Cloud deployment

This build is configured for Render via `render.yaml`: a Python FastAPI API service plus a React static site. Set the secret environment variables in Render before deploying. MongoDB should be a managed MongoDB/Atlas deployment.

### Payments

Billing uses Dodo Payments Checkout Sessions. Configure `DODO_PAYMENTS_API_KEY`, `DODO_GROWTH_MONTHLY_PRODUCT_ID`, `DODO_GROWTH_ANNUAL_PRODUCT_ID`, and `DODO_PAYMENTS_WEBHOOK_KEY`. Register `https://<your-api-domain>/api/webhook/dodo-payments` in Dodo. Dodo's hosted Checkout Session returns a checkout URL; payment state is synchronized by signed webhooks.

### Agent memory

Breeth is the persistent memory provider. MemoryOS exposes `/api/memory/write` and `/api/memory/search`, maps the default Breeth `group_id` to the authenticated organization, and keeps governance/audit state in MongoDB. Configure `BREETH_API_KEY` and optionally `BREETH_BASE_URL`.


## EmberGround hackathon fast path

The current build can run without MongoDB. Leave `MONGO_URL` empty and the backend uses an in-process Mongo-compatible store. Optional integrations:

- **n8n** — operational automation from governance decisions (`N8N_WEBHOOK_URL`)
- **Dodo Payments** — checkout and billing (`DODO_PAYMENTS_*`)
- **Breeth** — semantic agent memory (`BREETH_API_KEY`)

See `HACKATHON_1_HOUR_RUNBOOK.md` for the judge demo flow.
