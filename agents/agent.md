draw.io
# agent.md - Coding Agent Deployment Playbook (UberEats DIPaaS)

This runbook gives a coding agent a deterministic sequence to provision and deploy the Decision Intelligence Platform as a Service (DIPaaS) on AWS.

## 1) Objective

Deploy a multi-agent, production-grade architecture that:

1. Understands user intent from chat and voice.
2. Finds best restaurant outcomes (value + ETA + wait-time + preference fit).
3. Negotiates and optimizes cart outcomes.
4. Handles substitutions for 86'd items and upsell logic.
5. Applies restaurant promotions while maximizing customer savings and business profit.

## 2) Core Stack

- Edge/API: API Gateway + WAF + Shield
- Compute: Lambda
- Orchestration: Amazon Bedrock multi-agent routing
- Knowledge: Bedrock Knowledge Bases backed by OpenSearch + S3
- Data: Aurora PostgreSQL + S3 data lake
- Eventing: EventBridge ("Evenbridge" equivalent), SQS, optional Kinesis
- Security: IAM least privilege, KMS CMKs, Secrets Manager, VPC private subnets
- Ops: CloudWatch Logs/Metrics/Alarms + X-Ray traces

## 2.1) Service Options and Why Chosen

- API Layer
	- Chosen: Amazon API Gateway
	- Other options identified: Application Load Balancer + ECS/EKS ingress, AWS App Runner, direct CloudFront + Lambda URL
	- Why this fits best: Native throttling, auth controls, usage plans, and request transformation make it ideal for high-volume multi-channel APIs.

- Compute Runtime
	- Chosen: AWS Lambda
	- Other options identified: Amazon ECS on Fargate, Amazon EKS, AWS App Runner
	- Why this fits best: Event-driven scaling, low operational overhead, and strong integration with EventBridge, SQS, and Bedrock for bursty traffic.

- Multi-Agent Orchestration
	- Chosen: Amazon Bedrock Agents
	- Other options identified: Step Functions + custom orchestration, ECS-hosted orchestration service, third-party orchestration frameworks
	- Why this fits best: Managed orchestration with native model/tool integration and lower time-to-delivery for enterprise GenAI workflows.

- Knowledge and Retrieval
	- Chosen: Bedrock Knowledge Bases + OpenSearch + S3
	- Other options identified: Aurora pgvector, DynamoDB + vector extension patterns, external vector databases
	- Why this fits best: Managed RAG pipeline with enterprise governance, scalable vector + keyword retrieval, and durable low-cost source storage.

- Transactional Data Store
	- Chosen: Amazon Aurora PostgreSQL
	- Other options identified: Amazon DynamoDB, Amazon RDS PostgreSQL, Amazon Redshift
	- Why this fits best: Strong relational consistency for order/session workflows plus SQL flexibility for operational analytics.

- Event Backbone
	- Chosen: Amazon EventBridge + SQS, optional Kinesis
	- Other options identified: SNS + SQS only, Amazon MSK (Kafka), direct Lambda-to-Lambda
	- Why this fits best: Clean domain event routing, resilient decoupling, replay-friendly patterns, and optional streaming for very high throughput.

- Edge Security
	- Chosen: AWS WAF + AWS Shield
	- Other options identified: Third-party WAF/CDN stacks, API-only throttling controls
	- Why this fits best: Native DDoS and web threat protection tightly integrated with AWS edge and API services.

- Identity and Access
	- Chosen: AWS IAM
	- Other options identified: External policy engines, coarse account-level roles
	- Why this fits best: Fine-grained least-privilege controls and broad service-level integration.

- Encryption and Keys
	- Chosen: AWS KMS
	- Other options identified: Application-managed keys, CloudHSM-only approach
	- Why this fits best: Centralized key lifecycle management with native encryption integration across data services.

- Secrets Management
	- Chosen: AWS Secrets Manager
	- Other options identified: SSM Parameter Store only, application config files
	- Why this fits best: Rotation support, auditability, and secure runtime retrieval for credentials and tokens.

- Networking Boundary
	- Chosen: Amazon VPC private subnet model
	- Other options identified: Public-only service topology, hybrid-only perimeter model
	- Why this fits best: Strong network isolation, controlled east-west traffic, and compliance-friendly segmentation.

- Observability
	- Chosen: CloudWatch + AWS X-Ray
	- Other options identified: OpenTelemetry with self-hosted observability stack, third-party APM-only stacks
	- Why this fits best: Native metrics/logs/traces and lower operational complexity for production operations.

## 3) Prerequisites

- AWS account with Organization guardrails configured
- Authenticated AWS access in the target environment
- Python 3.11+
- IaC toolchain available in the delivery environment

## 4) Repository Layout

- `infra/`: infrastructure as code assets
- `src/`: service source code
- `deploy/`: deployment orchestration assets

## 5) Libraries

- boto3
- botocore
- pydantic
- fastapi
- mangum
- opensearch-py
- psycopg
- uvicorn

## 6) Terraform Module Families

- terraform-aws-modules/vpc/aws
- terraform-aws-modules/security-group/aws
- terraform-aws-modules/rds-aurora/aws
- terraform-aws-modules/s3-bucket/aws
- terraform-aws-modules/eventbridge/aws
- terraform-aws-modules/lambda/aws
- aws-ia/apigateway-v2/aws
- aws-ia/bedrock/aws (or equivalent Bedrock-supported module set)
- OpenSearch provider-based modules/resources

## 7) Deployment Sequence

1. Provision network and private connectivity boundaries.
2. Provision security controls and encryption baseline.
3. Provision persistent data plane (S3, Aurora, OpenSearch).
4. Provision eventing/streaming plane (EventBridge, SQS, Kinesis).
5. Provision application plane (API Gateway, Lambda ingress, agent Lambdas).
6. Provision Bedrock multi-agent orchestration and Knowledge Base integration.
7. Provision observability and operational guardrails.

## 8) Speech and Concierge Workflow

1. Ingest voice from Alexa channel.
2. Convert voice to text.
3. Route intent to Bedrock multi-agent orchestrator.
4. Execute intent, ranking, negotiation, substitution/upsell, and promotions agents.
5. Retrieve grounding context from Knowledge Base with OpenSearch and S3.
6. Persist decision and session outputs to Aurora/S3.
7. Generate response text and convert to speech.
8. Return spoken response to Alexa channel.

## 9) Scale and Reliability Requirements

- Support millions of requests through async fan-out and buffering.
- Enforce idempotency for all order-impacting operations.
- Apply retries and dead-letter routing for failed events.
- Configure autoscaling and concurrency controls for ingress and workers.

## 10) Security-in-Depth Requirements

- WAF and Shield protections at the edge.
- IAM least privilege and scoped trust boundaries.
- KMS encryption for data at rest and key management governance.
- Secrets Manager for runtime secret retrieval.
- Private subnet isolation for stateful services.
- Audit and threat detection coverage.

## 11) Observability Requirements

- Central logs, service metrics, alarms, and dashboards.
- Distributed traces for end-to-end workflow visibility.
- Business KPIs focused on decision friction and conversion lift.

## 12) CI/CD Requirements

1. Linting and unit validation.
2. Security scanning for app and IaC.
3. Plan/synth review gates.
4. Controlled promotion to production.
5. Post-deploy smoke and regression validation.

## 13) Validation Checklist

- Voice flow completes from speech input to spoken output.
- Agent workflow returns recommendation and cart optimization decisions.
- Knowledge retrieval is grounded and traceable.
- Event fan-out and asynchronous processing behave as expected.
- Security controls are active and monitored.
- Observability dashboards expose latency, errors, throttles, and business KPIs.

## 14) Rollback Strategy

- Versioned runtime releases with safe rollback targets.
- Progressive delivery and rollback at service boundaries.
- Data protection and recovery posture for persistent stores.

## 15) Instructions to Coding Agent (Prompt-Only)

Use the following short, user-friendly prompt set directly with a coding agent for a 45-60 minute interview.

Prompt 1 (Build the platform): "Set up a production-ready AWS Decision Intelligence platform for UberEats. Include API Gateway, Lambda, Bedrock multi-agent orchestration, Bedrock Knowledge Base with OpenSearch and S3, Aurora PostgreSQL, EventBridge, and SQS/Kinesis, with core security services in place (WAF, Shield, IAM, KMS, Secrets Manager, VPC). Also implement the full Alexa voice journey from speech input to spoken response."

Prompt 2 (Make it resilient and secure): "Make the system reliable at very high traffic scale with async processing, retries, DLQ, idempotency, and concurrency controls. Apply security-in-depth, and add monitoring/tracing with CloudWatch and X-Ray. Track business KPIs like conversion improvement and reduced decision friction."

Prompt 3 (Ship it safely): "Add CI/CD with quality checks, security scans, deployment approvals, environment promotion, smoke tests, and post-deploy validation. Add a clear rollback plan (versioning, canary or blue/green, and data recovery). Return final deployment outputs: endpoints, Bedrock agent and KB IDs, enabled security controls, scaling setup, known risks, and a prioritized remediation list."
