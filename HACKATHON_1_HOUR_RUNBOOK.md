# MemoryOS — EmberGround 1-Hour Demo Runbook

## The problem
AI agents are moving from answering questions to taking actions: reading customer data, modifying records, deploying software, and initiating financial operations. Authentication answers **who** the agent is; it does not decide whether a specific action is safe or permitted **right now**.

## The solution
MemoryOS is a runtime governance control plane:

**Identity → Risk → Policy → Effect → Audit**

Every action receives one enforceable effect:

- ALLOW — continue
- BLOCK — stop
- MODIFY — sanitize/constrain then continue
- ESCALATE — require human approval

## 60-second judge demo
1. Open `/playground`.
2. Run **Support bot reads public docs** → ALLOW.
3. Run **CRM copilot reads a customer** → MODIFY (PII redaction).
4. Run **DevOps copilot deletes production** → BLOCK.
5. Run **Finance agent transfers funds** → ESCALATE.
6. Show the live decision stream / audit trace.
7. Explain that the same decision is optionally sent to n8n for operational automation.
8. Open Pricing and show Dodo checkout as the monetization path.

## Zero-database demo mode
If `MONGO_URL` is blank, the backend uses `mongomock-motor` and keeps state in-process. This is intended for the hackathon demo. For persistent production state, set a real MongoDB URL later.

## Optional integrations
- n8n: `N8N_WEBHOOK_URL`
- Dodo Payments: Dodo API key/product IDs/webhook key
- Breeth: `BREETH_API_KEY`

The governance runtime remains usable when optional integrations are not configured.
