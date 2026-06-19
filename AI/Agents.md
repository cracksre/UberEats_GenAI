# Agents.md Coding Agent Build & Design Guide

Project: Uber Eats Decision Intelligence Platform as a Service (DIPaaS) â€” serverless GenAI ordering system combining Amazon Bedrock (Nova Sonic voice, Nova Canvas imagery), FastAPI/Lambda backend (Python 3.11+), and Vite/React frontend (TypeScript).

---

## User Stories

### US-1: Infrastructure and Database Deployment

As a platform engineer,
I want a single Terraform apply to provision the full network, security, and data plane (VPC, IAM roles, Cognito user pool with AppUserGroup, KMS CMKs, WAF + Shield on API Gateway and CloudFront, GuardDuty detector, CloudTrail, SQS dead-letter queues, and all five DynamoDB tables: Menu, Loyalty, Cart, Order, Chat),
so that every subsequent backend and GenAI service has a hardened, encrypted, auditable foundation with no manual console steps.

Acceptance criteria:
- `terraform apply` completes with zero errors from a clean state; `terraform plan` shows no drift on re-run.
- All five DynamoDB tables exist with KMS CMK encryption; a test `PutItem` and `GetItem` round-trip succeeds on each.
- WAF WebACL is attached to both API Gateway stage and CloudFront distribution; a simulated SQLi request returns HTTP 403.

Do NOT:
- Do not use `AdministratorAccess` or wildcard `"*"` actions in any IAM role; scope every execution role to the minimum required actions and resources.
- Do not store Terraform state locally; remote state in S3 with DynamoDB locking is required before first apply.
- Do not apply `terraform apply` directly against production without first running `terraform plan -out=tfplan` and having the plan reviewed.

---

### US-2: UI Deployment

As a customer using voice or web,
I want the Vite/React SPA (UbereatsFrontendUI) deployed to AWS Amplify with Cognito sign-in, CloudFront-served menu images, and a working Nova Sonic bidirectional WebSocket session,
so that I can authenticate, browse the AI-generated menu with real food photography, and place a voice order end-to-end without leaving the browser.

Acceptance criteria:
- Amplify build succeeds from the `main` branch; the public URL returns HTTP 200 and renders `<div id="root">`.
- Cognito sign-in issues a valid JWT; the JWT is accepted by the API Gateway Cognito authorizer on `GET /menu`.
- Nova Sonic WebSocket session establishes over SigV4-signed bidirectional stream; a spoken item name triggers `AddToCart` and the cart total updates.

Do NOT:
- Do not hardcode `userPoolId`, `userPoolClientId`, `menuAPIURL`, or `loyaltyAPIURL` in the JS bundle or in `config/default-settings.json` committed to source control; inject them as Amplify environment variables at build time.
- Do not expose raw S3 URLs for menu images in the UI; all image references must route through the CloudFront domain.
- Do not use `uvicorn` in the Amplify build pipeline; it is a local development tool and must never be deployed.

---

### US-3: GenAI Agent Deployment

As a customer with decision friction,
I want the Bedrock multi-agent orchestrator (Intent, Ranking, Negotiation, Substitution/Upsell, Promotions agents) deployed as FastAPI + Mangum Lambda functions with action groups wired to `POST /conversation`, `POST /recommend`, and `POST /order`,
so that when I speak a vague request ("something spicy, under $20, quick") the platform understands my intent, ranks options from the Knowledge Base, negotiates the best cart outcome, handles 86'd items, and applies the highest-value promotion â€” all within a single voice turn.

Acceptance criteria:
- All five agent Lambdas deploy without error; X-Ray active tracing shows an end-to-end trace from API Gateway through Bedrock Agent to DynamoDB on each invocation.
- `POST /conversation` with input `"I want spicy food under $20"` returns a ranked recommendation grounded in the OpenSearch Knowledge Base with a source citation.
- Substitution Agent returns an alternative item when a test item is marked unavailable in the Menu table; Promotion Agent applies an eligible offer before `SubmitOrder` is called.

Do NOT:
- Do not invoke `SubmitOrder` before `GetCurrentCartItems` confirms the cart total; the agent must never self-calculate prices.
- Do not call `FinalizeSessionForNextCustomer` before `SubmitOrder` returns a success response; premature session reset causes unrecoverable order data loss.
- Do not grant Lambda execution roles `bedrock:*` wildcard permissions; scope to `bedrock:InvokeAgent` and `bedrock:Retrieve` on specific ARNs only.

---

## Build & Test Commands

Frontend (src/UbereatsFrontendUI/):
```bash
npm -v
npm install -g npm@latest && npm run build    # Build Vite SPA (output: dist/)
npm run dev                     # Local dev server on :5173
npm run preview                 # Preview production build
```

Backend (agents/):
```bash
python -m pytest tests/ -v                    # Full test suite with coverage
python -m pytest tests/test_intent_agent.py   # Single agent test
python -m pytest --mock-aws                   # Mock all boto3/DynamoDB calls
pip install -e ".[dev]"                       # Install editable + dev dependencies
```

Python Libraries (requirements.txt):
```
boto3              		# AWS SDK — Bedrock, DynamoDB, S3, Cognito, SQS
botocore           		# AWS HTTP transport, SigV4 signing, error handling
fastapi            		# HTTP endpoints for agent Lambdas
pydantic           		# Request/response validation
opensearch-py      		# OpenSearch Serverless queries (Ranking, Substitution agents)
aws-lambda-powertools  	# Structured logging, tracing, middleware
moto[all]          		# AWS service mocks for unit tests (dev only)
pytest             		# Test runner (dev only)
```

Infrastructure (infra/terraform/):
Create a terraform main.tf with the following terraform modules:
### Network and Edge

- terraform-aws-modules/vpc/aws
- terraform-aws-modules/security-group/aws

### Compute and API

- terraform-aws-modules/lambda/aws
- aws-ia/apigateway-v2/aws

### Storage and Data

- terraform-aws-modules/s3-bucket/aws
- terraform-aws-modules/dynamodb-table/aws

### Identity and Access

- terraform-aws-modules/iam/aws
  - Submodules: `iam-role`, `iam-policy`, `iam-assumable-role`
- terraform-aws-modules/cognito-user-pool/aws

### Encryption

- terraform-aws-modules/kms/aws

### Security and Edge Protection

- hashicorp/aws provider — `aws_wafv2_web_acl`, `aws_wafv2_rule_group`, `aws_wafv2_ip_set`
- hashicorp/aws provider — `aws_shield_protection`, `aws_shield_protection_group`
  - Note: Shield Advanced requires prior subscription activation via the AWS console or CLI
- hashicorp/aws provider — `aws_guardduty_detector`, `aws_guardduty_filter`, `aws_guardduty_publishing_destination`

### Content Delivery

- terraform-aws-modules/cloudfront/aws

### Messaging and Buffering

- terraform-aws-modules/sqs/aws
  - Used for async fan-out between agent Lambdas and dead-letter routing for failed events

### Observability and Audit

- hashicorp/aws provider — `aws_cloudtrail`, `aws_cloudtrail_event_data_store`
  - Requires S3 bucket with CloudTrail-specific bucket policy; use terraform-aws-modules/s3-bucket/aws with the `attach_cloudtrail_policy` flag
- hashicorp/aws provider — `aws_xray_group`, `aws_xray_sampling_rule`, `aws_xray_encryption_config`
  - X-Ray tracing is enabled per Lambda via `tracing_config { mode = "Active" }` in the Lambda module

### AI and Knowledge

- aws-ia/bedrock/aws (or equivalent Bedrock-supported module set)
- OpenSearch provider-based modules/resources


## Code Style Guidelines

- Python: PEP 8 enforced via `black src/ --line-length 100` and `flake8 src/ --max-line-length=100`
- Frontend: Prettier + ESLint; run `npm run lint:fix` before commit
- Terraform: `terraform fmt -recursive infra/` before planning
- Commit messages: `<type>(<scope>): <subject>` (e.g., `feat(intent-agent): add fallback intent`)

## Testing Instructions

- Mock AWS services: Use `moto` library; decorate tests with `@mock_dynamodb`, `@mock_bedrock`
- Secrets in tests: Never hardcode API keys; use environment fixtures: `os.environ['BEDROCK_AGENT_ID'] = 'test-id'`
- Frontend unit tests: Vitest; run `npm run test`

## Security Considerations

- Never commit: `.env`, `aws-config.json`, Terraform variable files with credentials, SSH keys
- Secrets handling: All secrets stored in AWS Secrets Manager; retrieve at runtime via boto3, not hardcoded
- KMS encryption: All DynamoDB tables encrypted with customer-managed CMK; Lambda execution roles assume encryption permissions via IAM
- No plaintext logs: Never log request/response bodies containing PII or payment data

## Commit & PR Guidelines

- Branch naming: `feature/agent-name`, `fix/bug-description`, `infra/service-name`, `docs/topic`
- Merge strategy: Squash commits to `main`; rebase branches before PR
- PR checklist: Unit tests pass 
- `terraform plan` clean (infra changes)
- no hardcoded secrets 
- CHANGELOG.md updated
- Code review: At least 2 approvals before merge 
- automated security scanning (Snyk) must pass


- Create a naming.tf with all the parameters constant for the environment and the services:

 - region = "east=us-1"
 - env = "dev"
 - app_id = "app-1"
 - cmdb_id = "CMDB0001"

```terraform pipeline (iac-pipeline.yaml):
Create a terraform pipeline to refer the main.tf and naming.tf to deploy the infrastructure in AWS. Use the terraform modules listed above.

In the first stage install the tools:
 - github.exe
 - python.exe
 - terraform.exe

Initialize terraform using terraform init 

 - Run terraform Plan terraform plan -out=tfplan

 - Run  terraform apply tfplan ONLY if the env = dev or env = test.
 - DO NOT run terraform apply tfplan if env = prod
 - Run terraform destroy -auto-approve  # Dev teardown only
```
## CI/CD (BUild/Test/Deploy) - PaaS Deployment

# Create a CI with stages: (pipeline-ci.yaml)
- Build (Refer to Python Libraries in (requirements.txt))
- Test (Specific testing instructions are provided in the Testing Instructions section)
	- Unit test using Playwright
	- Code Quality Test using SonarQube (Refer Section ## Code Style Guidelines)
	- SCA, SAST using SNYK (Refer details in section ## Security Considerations)
	- DAST using Rapid7
- Package
- Publish Package as Artifact to Artifactory

# Create a CD pipeline with stages: (pipeline-cd.yaml)
- Only utilize the tag mentioned in the Artifactory with lifecycle = lta or release for deployment.
Following stages should run:
- Pull artifact from Artifactory
- Unzip if already zipped
- Deploy to Amplify / API Gateway / Lamda 

