# Uber Eats — Decision Intelligence Platform as a Service (DIPaaS)

A GenAI-powered voice ordering and decision intelligence system built on AWS. The platform eliminates **Decision Friction** — the hesitation customers experience after discovery — by acting as a concierge that understands intent, ranks options, negotiates cart outcomes, substitutes unavailable items, and applies promotions in real time.

---

## Repository Structure

```
AI/
├── README.md                     ← this file
├── Agents.md                     ← coding agent deployment playbook
├── UberEats.drawio               ← architecture diagram (foundation + DIPaaS layers)
├── deploy/                       ← deployment orchestration (to be populated)
├── infra/                        ← Terraform IaC modules (to be populated)
└── src/
    └── UbereatsFrontendUI/       ← compiled Vite/React SPA (deployed to AWS Amplify)
        ├── index.html
        ├── assets/               ← bundled JS, CSS, audio worklet
        ├── config/
        │   └── default-settings.json   ← Cognito + agent system prompt + tool definitions
        └── samples/
            ├── samples-index.json
            ├── uber-eats/        ← primary sample (voice ordering concierge)
            ├── coffee-shop/
            ├── hotel-room-service/
            └── pizza-delivery/
```

---

## Architecture Overview

The platform is built in two layers stacked on the same AWS account.

### Layer 1 — Foundation (UberEats.drawio)

Establishes the base platform before GenAI orchestration is applied.

| Component | Service | Purpose |
|---|---|---|
| Auth | Amazon Cognito | User pools, AppUserGroup (`/appuser`), STS token vending, JWT issuance |
| Edge | API Gateway | Single entry point for all client channels |
| CDN | Amazon CloudFront | Serves AI-generated menu images from S3 |
| Storage | Amazon S3 | Menu image origin + data lake |
| Data | Amazon DynamoDB | Menu, Loyalty, Cart, Order, Chat tables |
| Compute | AWS Lambda | Data and Image Function — seeds menu, calls Nova Canvas for food photography |
| AI Images | Amazon Bedrock (Nova Canvas) | Generates professional food photography per menu item |
| Frontend | AWS Amplify | Hosts and CI/CD deploys the SPA |
| Security | WAF + Shield + GuardDuty | Edge protection and continuous threat detection |

### Layer 2 — DIPaaS Intelligence (UberEats.drawio + Agents.md)

Layered on top of the Foundation. Every customer interaction becomes a decision point.

| Component | Service | Purpose |
|---|---|---|
| Orchestrator | Amazon Bedrock Agents + AgentCore | Multi-agent routing and session management |
| Intent Agent | Bedrock Agent + Lambda | Parses customer utterance into structured intent |
| Ranking Agent | Bedrock Agent + Lambda | Scores restaurants by value, ETA, wait time, preference fit |
| Negotiation Agent | Bedrock Agent + Lambda | Optimises pricing and offer scenarios for the cart |
| Substitution/Upsell Agent | Bedrock Agent + Lambda | Handles 86'd items, recommends alternatives, upsells |
| Promotions Agent | Bedrock Agent + Lambda | Applies eligible offers while maximising savings and margin |
| Knowledge | Bedrock Knowledge Bases + OpenSearch Serverless + S3 | RAG retrieval — grounding agent responses in real restaurant data |
| Memory | Agent Memory Store | Maintains session context across multi-turn conversations |
| Encryption | AWS KMS | CMKs for data at rest across all services |
| Observability | CloudWatch + AWS X-Ray | Logs, metrics, alarms, distributed traces |

---

## End-to-End Workflow

```
Customer speaks via Alexa / Web UI / Mobile / Kiosk
         │
         ▼
[ Cognito JWT issued ] ──► API Gateway (WAF + Shield at edge)
         │
         ▼
    Lambda Ingress
         │
         ▼
  Bedrock Agent Orchestrator
    ├── Intent Agent          ← understands what the customer actually wants
    ├── Ranking Agent         ← finds the best restaurant match
    ├── Negotiation Agent     ← optimises the cart value
    ├── Substitution/Upsell   ← handles 86'd items, suggests better alternatives
    └── Promotions Agent      ← applies offers, maximises savings
         │
         ├── Bedrock Knowledge Bases
         │       ├── OpenSearch Serverless  ← vector + keyword retrieval
         │       └── S3 data lake          ← source documents
         │
         ├── DynamoDB          ← Menu, Cart, Order, Loyalty, Chat tables
         │
         └── Merchant/POS APIs ← live menu, stock, ETA confirmation
         │
         ▼
    Amazon Polly (text → speech)
         │
         ▼
    Spoken response returned to customer channel
```

### Frontend Interaction Detail

The SPA (`UbereatsFrontendUI`) uses **Amazon Bedrock Runtime `InvokeModelWithBidirectionalStream`** over WebSocket for real-time voice interaction with **Amazon Nova Sonic**. The agent tools are JavaScript functions defined inside `settings.json` and executed client-side, calling the backend APIs with a Cognito JWT.

| Tool | What it does | Backend call |
|---|---|---|
| `GetDateAndTime` | Returns current timestamp for agent context | Client-side only |
| `GetCustomerLoyaltyInfo` | Looks up loyalty points by phone number | `GET {loyaltyAPIURL}?phone=…` with JWT |
| `GetMenuItems` | Fetches full menu catalogue (cached 2 min) | `GET {menuAPIURL}` with JWT |
| `AddToCart` | Adds item + customisations to the cart component | Client-side cart state |
| `addCustomizationToCartItem` | Applies customisations to an existing cart line | Client-side cart state |
| `RemoveItemFromCart` | Removes item by ID or natural language description | Client-side cart state |
| `GetCurrentCartItems` | Returns live cart state and running total | Client-side cart state |
| `ShowCategoryItems` | Filters the visible menu to a category (upsell strategy) | Client-side UI component |
| `GetCategoryList` | Returns all available categories | Client-side menu component |
| `SubmitOrder` | Posts the confirmed cart to the order API | `POST /order` with JWT |
| `FinalizeSessionForNextCustomer` | Resets session state after order is complete | Client-side session reset |

The tools that touch the backend always:
1. Call `auth.getTokens()` to obtain the Cognito `idToken`.
2. Pass the token as `Authorization: {idToken}` on every API Gateway request.

---

## Python Libraries — Where Each One Lives

The Python libraries listed in `Agents.md §5` are **backend-only**. The frontend is JavaScript (Vite/React). Here is the precise mapping:

| Library | Used in | Purpose |
|---|---|---|
| `boto3` | All agent Lambdas + Data and Image Function | AWS SDK — calls Bedrock Agent Runtime, DynamoDB, S3, Cognito, OpenSearch, Polly |
| `botocore` | All agent Lambdas | Low-level AWS HTTP transport, SigV4 signing, retry logic; `botocore.exceptions.ClientError` is caught on every AWS call |
| `pydantic` | All agent Lambdas | Validates and serialises request/response payloads (`/conversation`, `/recommend`, `/order`) |
| `fastapi` | All agent Lambdas | Web framework that implements the Lambda-hosted HTTP endpoints |
| `mangum` | All agent Lambdas | ASGI adapter — wraps the FastAPI app so API Gateway events are handled inside Lambda |
| `opensearch-py` | Ranking Agent, Substitution Agent | Queries OpenSearch Serverless vector index for restaurant and menu retrieval; uses AWS4Auth (boto3-based) — **not** username/password |
| `uvicorn` | Local development only | ASGI server for running FastAPI locally before deployment; **never deployed to Lambda** |

### Typical Lambda handler pattern

```python
from fastapi import FastAPI, Depends
from mangum import Mangum
from pydantic import BaseModel
import boto3
import botocore

app = FastAPI()

class OrderRequest(BaseModel):
    session_id: str
    utterance: str

@app.post("/conversation")
def conversation(req: OrderRequest):
    client = boto3.client("bedrock-agent-runtime")
    try:
        response = client.invoke_agent(
            agentId="...",
            agentAliasId="...",
            sessionId=req.session_id,
            inputText=req.utterance,
        )
        return {"response": response}
    except botocore.exceptions.ClientError as e:
        raise

handler = Mangum(app)  # Lambda entry point
```

```bash
# Local testing only — not deployed
uvicorn main:app --reload --port 8000
```

---

## Agent Lambdas — Backend Functions to Provision

```
src/
└── agents/
    ├── intent_agent.py          # POST /conversation → Bedrock orchestrator
    ├── restaurant_agent.py      # restaurant ranking via OpenSearch + DynamoDB
    ├── negotiation_agent.py     # cart value optimisation
    ├── substitution_agent.py    # 86'd item handling + upsell
    └── upsell_agent.py          # promotion and combo detection
```

API Gateway endpoints:

| Method | Path | Handler | Description |
|---|---|---|---|
| `POST` | `/conversation` | `intent_agent` | Receives utterance, routes to Bedrock orchestrator |
| `POST` | `/recommend` | `restaurant_agent` | Returns ranked restaurant list |
| `POST` | `/order` | _(order Lambda)_ | Submits confirmed cart to Order table |
| `GET` | `/menu` | _(menu Lambda)_ | Returns full menu catalogue |
| `GET` | `/loyalty` | _(loyalty Lambda)_ | Returns loyalty points by phone |

All endpoints require a valid Cognito JWT in the `Authorization` header, validated by the API Gateway Cognito authorizer.

---

## Frontend Configuration

Before deploying the SPA to Amplify, populate `config/default-settings.json` and the sample `settings.json` files with live values:

```json
{
  "settings": {
    "cognito": {
      "userPoolId": "us-east-1_XXXXXXXXX",
      "userPoolClientId": "XXXXXXXXXXXXXXXXXXXXXXXXXX",
      "identityPoolId": "us-east-1:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "region": "us-east-1"
    },
    "agent": {
      "agentId": "XXXXXXXXXX",
      "agentAliasId": "XXXXXXXXXX",
      "region": "us-east-1"
    },
    "globals": {
      "menuAPIURL": "https://{api-id}.execute-api.us-east-1.amazonaws.com/prod/menu",
      "loyaltyAPIURL": "https://{api-id}.execute-api.us-east-1.amazonaws.com/prod/loyalty"
    }
  }
}
```

---

## Deployment Sequence

Follow the sequence in `Agents.md §7` in strict order to avoid dependency failures:

1. **Network** — VPC, private subnets, security groups
2. **Identity + Encryption** — IAM roles (least privilege per Lambda), Cognito user pool + AppUserGroup, KMS CMKs
3. **Data plane** — S3 buckets, DynamoDB tables (Menu, Loyalty, Cart, Order, Chat), OpenSearch Serverless collection
4. **Compute** — Lambda Data and Image Function (seeds menu + calls Nova Canvas), agent Lambdas
5. **Knowledge** — Bedrock Knowledge Base connected to OpenSearch + S3
6. **Orchestration** — Bedrock Agent + AgentCore, action groups wired to agent Lambdas
7. **API** — API Gateway with Cognito authorizer, WAF association
8. **Frontend** — Amplify app connected to Git repo, environment variables injected, build triggered
9. **Observability** — CloudWatch dashboards, X-Ray traces, alarms on latency / token usage / conversion / abandonment

---

## Security Controls

| Control | Service | Coverage |
|---|---|---|
| Edge protection | WAF + Shield | DDoS and OWASP Top 10 at API Gateway and CloudFront |
| Authentication | Amazon Cognito | JWT issuance, group-based access (`AppUserGroup`) |
| API authorisation | API Gateway Cognito Authorizer | Every backend endpoint validates JWT before invocation |
| Service permissions | AWS IAM | Least-privilege execution role per Lambda; no wildcard actions |
| Data encryption | AWS KMS | CMKs for DynamoDB, S3, OpenSearch at rest |
| Network isolation | VPC private subnets | Stateful services unreachable from the public internet |
| Threat detection | Amazon GuardDuty | Continuous behavioural anomaly detection across the account |

---

## Success Metrics

| Metric | Description |
|---|---|
| Decision Time Reduction | Average time from first utterance to confirmed order |
| Conversion Rate | Percentage of sessions that result in a submitted order |
| Average Order Value | Revenue per completed order |
| Merchant Revenue Lift | Incremental revenue attributable to upsell and negotiation agents |
| Customer Satisfaction | Post-order rating and repeat session rate |
