# MemoryOS + n8n

MemoryOS optionally POSTs every governance decision to `N8N_WEBHOOK_URL`. Governance does **not** depend on n8n being available.

## 1. Import
Import `MemoryOS-Agent-Decision-Router.json` into n8n. Activate it and copy the Production Webhook URL.

## 2. Configure MemoryOS
Set:

```env
N8N_WEBHOOK_URL=https://YOUR-N8N-HOST/webhook/memoryos-decision
```

## 3. Demo
Run the public playground scenarios:

- `block` → n8n routes to Security Alert
- `escalate` → n8n routes to Human Review
- `modify` → n8n routes to Safe Payload
- `allow` → n8n routes to Continue

For the hackathon, connect the BLOCK/ESCALATE branches to Slack, email, Jira, Teams, or another operational system if credentials are available.
