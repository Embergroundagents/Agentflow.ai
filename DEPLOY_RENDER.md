# MemoryOS — Render + Dodo Payments + Breeth

## 1. Deploy on Render

The repository includes `render.yaml` with two services:

- `memoryos-api` — FastAPI backend
- `memoryos-web` — React static site

Create/connect the repository in Render and deploy the Blueprint.

### Required API secrets

Set these in the `memoryos-api` service:

```text
MONGO_URL (optional — omit for in-memory demo mode)=<MongoDB Atlas connection string>
ADMIN_EMAIL=<admin email>
ADMIN_PASSWORD=<strong admin password>
DODO_PAYMENTS_API_KEY=<Dodo API key>
DODO_GROWTH_MONTHLY_PRODUCT_ID=<Dodo monthly product id>
DODO_GROWTH_ANNUAL_PRODUCT_ID=<Dodo annual product id>
DODO_PAYMENTS_WEBHOOK_KEY=<Dodo webhook signing key>
BREETH_API_KEY=<Breeth API key>
```

`JWT_SECRET` is generated automatically by the Render Blueprint.

For the frontend, `REACT_APP_BACKEND_URL` points to the API service URL. If you use a custom domain, update both `REACT_APP_BACKEND_URL` and the backend `ALLOWED_ORIGINS` value.

## 2. Configure Dodo Payments

Create the Growth monthly and annual products in Dodo Payments and copy their product IDs into Render.

Register this webhook endpoint in the Dodo dashboard:

```text
https://<your-api-domain>/api/webhook/dodo-payments
```

Enable the payment/subscription events required by your product. MemoryOS verifies the signed webhook, deduplicates by `webhook-id`, and updates the local payment transaction state.

The checkout route is:

```text
POST /api/payments/checkout
```

The frontend redirects to Dodo's hosted Checkout Session and returns to `/pricing/success`.

## 3. Configure Breeth

Create a Breeth project and API key, then set:

```text
BREETH_API_KEY=<key>
BREETH_BASE_URL=https://api.thebreeth.com/v1
```

MemoryOS exposes:

```text
POST /api/memory/write
POST /api/memory/search
```

The default Breeth `group_id` is automatically namespaced with the authenticated MemoryOS organization:

```text
<org_id>:default
```

Custom groups become:

```text
<org_id>:<custom_group>
```

This keeps memory tenant-scoped while allowing sub-scopes for customers, agents, projects, or workflows.

## 4. Architecture

```text
AI Agent
   |
   v
MemoryOS Runtime
   |-- Identity
   |-- Risk
   |-- Policy
   |-- Allow / Block / Modify / Escalate
   |-- Audit
   |
   +----> MongoDB       (governance + audit state)
   |
   +----> Breeth        (persistent semantic / intent-aware memory)
   |
   +----> Dodo Payments (SaaS billing)
   |
   +----> Enterprise connectors / APIs
```

## 5. Local development

Backend:

```bash
cd backend
cp .env.example .env
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn server:app --reload --host 127.0.0.1 --port 8001
```

Frontend:

```bash
cd frontend
yarn install
yarn start
```

The frontend expects:

```text
REACT_APP_BACKEND_URL=http://localhost:8001
```

Do not commit real Dodo or Breeth API keys.


## EmberGround demo mode
For the hackathon, MongoDB is optional. If `MONGO_URL` is not set, the API uses `mongomock-motor` and keeps state in the service process. This avoids database setup and is suitable for a short-lived demo. Set a real MongoDB URL later for persistence.

### n8n
Set `N8N_WEBHOOK_URL` to the Production Webhook URL from `integrations/n8n/MemoryOS-Agent-Decision-Router.json`. MemoryOS sends governance decisions to n8n asynchronously; a failed n8n workflow never blocks the governance decision.
