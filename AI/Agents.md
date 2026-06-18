# Uber Eats DIPaaS Deployment Agent

## Objective# Situation:  
Let’s take a situation. I am a customer trying to find restaurants in my area.
Most recommendations systems will go:
“Here are the listed restaurants in your area”.
At this stage the customer is confused if they should go for Korean, Chinese, Italian, American, etc.
Maybe the best restaurant shown by review was 40 mins away and there is a waiting time. And then the intent to purchase just hangs there.
For the business, it’s a lost opportunity. Maybe the customer and their family were hungry and that could mean a fat bill for the restaurant.
Here comes the BIG issue of “Decision Friction”, which is causing businesses cumulatively USD 1B+ in lost business opportunity in the Restaurant and Food delivery industry.

Most digital platforms obsess over :
•	Discovery 
•	Recommendations 
•	Search relevance 
•	Ads
But the real leakage is after discovery.
A customer finds:
•	20 restaurants on Uber Eats
•	Best reviewed restaurants on Yelp
•	Highest stars received in Zomato
Then they hesitate.
This hesitation creates:
•	Abandoned carts 
•	Lower conversion rates 
•	Lower order values 
•	Refunds 
•	Customer churn
Most companies measure clicks and purchases.
Very few measure:
•	"Why didn't the customer decide?"
•	This is a trillion-dollar inefficiency across digital commerce.

# Task:

Talk about the architecture to build the solution.

My Task is to create a Agents.md file to work with the coding agent and provide context of the architecture diagram to build the solution of a Decision Intelligence Platform to build and deploy the solution through Prompt engineering.


<!-- <!-- My task is to evolve the existing Alexa-based architecture into a true GenAI Decision Intelligence Platform as a Service (DIPaaS).

The current solution already has a voice-enabled customer experience and end-to-end Alexa flow, but it remains primarily a voice customer service path that delivers responses, rather than a decision intelligence platform that understands intent, negotiates outcomes, substitutes unavailable items, and optimizes promotions in real time.

The objective is to identify where the current architecture can be extended with GenAI so that the agent becomes a concierge and decision partner that:

1.	Understands customer intent beyond simple commands
2.	Finds the best restaurant and order outcomes
3.	Negotiates cart optimization and cost tradeoffs
4.	Substitutes 86’d items and upsells relevant alternatives
5.	Applies promotions while maximizing customer savings and business profit

This turns the experience from “Customer Service as a Service” into “Revenue Generation as a Service,” and from a single Alexa channel into a reusable platform that can support multiple applications. -->

In my mind I need to build a system which is a "Decision Intelligence Platform as a Service (DIPaaS)".

A centralized GenAI platform that serves multiple consumer applications (Uber Eats, Ticketmaster, Shopify) while allowing domain-specific agents and models. -->

# Solution:
The solution is designed as a GenAI-backed architecture layered on top of the existing Alexa voice and endpoint flow.

The current draw.io diagram captures the new DIPaaS architecture by layering Bedrock Agents, Knowledge Bases, event-driven orchestration, and decision-centric data stores on top of the Alexa input/output path. This architecture converts the legacy customer service workflow into a decision intelligence workflow by making every step of the UX a decision point rather than a static response.

# Architecture Transition: From Legacy Customer Service to Decision Intelligence

The designed solution should now include a second architecture diagram that shows:

- The old customer service system: Alexa voice input → speech-to-text → API gateway → request processing → static response generation → text-to-speech output.

- The new Decision Intelligence system: the same Alexa channel plus Bedrock multi-agent orchestration, intent/ranking/negotiation/substitution/promotion agents, knowledge retrieval and distribution(KB/CDN), event backbone, and DynamoDB to keep the cost low.

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

### Foundation Layer (UberEats.drawio)

The foundational AWS infrastructure establishes the base platform before any GenAI orchestration is layered on top. It includes:

- Auth: Amazon Cognito — user authentication with Cognito Groups (`AppUserGroup /appuser`), token exchange via AWS STS, and JSON Web Token issuance.
- Edge/API: API Gateway — routes all client requests from Alexa Skill, Mobile Device, WebApp, and Kiosk channels.
- CDN: Amazon CloudFront — serves menu images from S3 to all client channels with low-latency global distribution.
- Storage: Amazon S3 (Standard Bucket + delivery bucket) — stores AI-generated food photography and menu assets produced by the Lambda Data and Image Function.
- Data: Amazon DynamoDB — five domain tables: Menu, Loyalty, Cart, Order, and Chat.
- Compute: AWS Lambda (Data and Image Function) — populates the system with a complete embedded menu (entrees, main courses, soups and salads, breads, curries, and sweet dishes) and calls Amazon Nova Canvas to generate professional food photography for each item, storing images in S3 for CloudFront delivery.
- AI Image Generation: Amazon Bedrock — Amazon Nova Canvas model, invoked by the Lambda Data and Image Function to produce menu item photography automatically.
- Frontend Hosting: AWS Amplify — hosts the web application frontend.
- Security: AWS WAF + AWS Shield + Amazon GuardDuty — edge protection and threat detection.
- Network Boundary: AWS Region isolation with VPC account boundary (`VoiceAi Account`).

### DIPaaS Intelligence Layer

- Edge/API: API Gateway + WAF + Shield
- Compute: Lambda
- Orchestration: Amazon Bedrock multi-agent routing (Bedrock Agent + Bedrock AgentCore)
- Memory: Agent Memory store for session context
- Knowledge: Bedrock Knowledge Bases backed by OpenSearch + S3
- Data: DynamoDB (low-cost operational tables) + S3 data lake
- Identity and Access: Amazon Cognito (user pools + groups + STS/JWT) + AWS IAM (least-privilege roles and policies for every service)
- Security: KMS CMKs, VPC private subnets
- Ops: CloudWatch Logs/Metrics/Alarms + X-Ray traces

## 2.1) Service Options and Why Chosen

- Identity and Access Management
	- Chosen: Amazon Cognito + AWS IAM
	- Other options identified: AWS IAM Identity Center, Auth0 + IAM roles, custom JWT service with coarse account-level roles
	- Why this fits best: Cognito manages user-facing authentication (user pools, groups, STS token vending, JWT issuance) and integrates natively as an API Gateway authorizer; IAM enforces least-privilege execution roles and resource policies across every AWS service — together they cover both the human identity plane and the machine identity plane without additional tooling.

- Content Delivery
	- Chosen: Amazon CloudFront
	- Other options identified: AWS Global Accelerator, third-party CDN, direct S3 presigned URLs
	- Why this fits best: Low-latency global distribution of menu images with S3 origin integration, caching, and WAF attachment at the edge.

- Operational Data Store
	- Chosen: Amazon DynamoDB
	- Other options identified: Amazon Aurora PostgreSQL, Amazon RDS, Redis
	- Why this fits best: Single-digit millisecond reads for high-frequency cart, menu, and session tables at scale, with no server management and pay-per-request pricing that keeps costs low during variable traffic.

- AI Image Generation
	- Chosen: Amazon Bedrock — Amazon Nova Canvas
	- Other options identified: Stable Diffusion on SageMaker, DALL·E via third-party API, pre-shot photography workflow
	- Why this fits best: Fully managed generative image model within the AWS trust boundary; no third-party data egress, consistent IAM-governed access, and direct Lambda invocation for automated menu population.

- Frontend Hosting
	- Chosen: AWS Amplify
	- Other options identified: S3 static site + CloudFront, EC2/ECS-hosted frontend, Vercel/Netlify
	- Why this fits best: Managed CI/CD hosting for web application with built-in Cognito integration and branch-based deployment.

- Threat Detection
	- Chosen: Amazon GuardDuty
	- Other options identified: Third-party SIEM, CloudTrail-only monitoring
	- Why this fits best: Continuous automated threat detection across AWS accounts with minimal configuration and native integration into Security Hub.

- API Layer
	- Chosen: Amazon API Gateway
	- Other options identified: Application Load Balancer + ECS/EKS ingress, AWS App Runner, direct CloudFront + Lambda URL
	- Why this fits best: Native throttling, auth controls, usage plans, and request transformation make it ideal for high-volume multi-channel APIs.

- Compute Runtime
	- Chosen: AWS Lambda
	- Other options identified: Amazon ECS on Fargate, Amazon EKS, AWS App Runner
	- Why this fits best: Event-driven scaling, low operational overhead, and strong integration with Bedrock for bursty traffic.

- Multi-Agent Orchestration
	- Chosen: Amazon Bedrock Agents
	- Other options identified: Step Functions + custom orchestration, ECS-hosted orchestration service, third-party orchestration frameworks
	- Why this fits best: Managed orchestration with native model/tool integration and lower time-to-delivery for enterprise GenAI workflows.

- Knowledge and Retrieval
	- Chosen: Bedrock Knowledge Bases + OpenSearch + S3
	- Other options identified: Aurora pgvector, DynamoDB + vector extension patterns, external vector databases
	- Why this fits best: Managed RAG pipeline with enterprise governance, scalable vector + keyword retrieval, and durable low-cost source storage.

- Operational Data Store
	- Chosen: Amazon DynamoDB
	- Other options identified: Amazon Aurora PostgreSQL, Amazon RDS PostgreSQL, Amazon Redshift
	- Why this fits best: Single-digit millisecond reads for high-frequency cart, menu, order, and session tables with no server management and pay-per-request pricing; the same DynamoDB tables provisioned in the Foundation Layer serve the full intelligence layer without an additional relational database.

- Edge Security
	- Chosen: AWS WAF + AWS Shield
	- Other options identified: Third-party WAF/CDN stacks, API-only throttling controls
	- Why this fits best: Native DDoS and web threat protection tightly integrated with AWS edge and API services.

- Identity and Access Management
	- Chosen: Amazon Cognito + AWS IAM
	- Other options identified: AWS IAM Identity Center, Auth0 + IAM roles, custom JWT with coarse account-level roles
	- Why this fits best: Cognito manages user-facing authentication (user pools, groups, STS token vending, JWT issuance) while IAM enforces least-privilege service-to-service permissions, execution roles, and resource policies across every AWS service in the stack — together they cover both the human identity plane and the machine identity plane.

- Encryption and Keys
	- Chosen: AWS KMS
	- Other options identified: Application-managed keys, CloudHSM-only approach
	- Why this fits best: Centralized key lifecycle management with native encryption integration across data services.

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
- uvicorn

## 6) Terraform Module Families

- terraform-aws-modules/vpc/aws
- terraform-aws-modules/security-group/aws
- terraform-aws-modules/s3-bucket/aws
- terraform-aws-modules/lambda/aws
- aws-ia/apigateway-v2/aws
- aws-ia/bedrock/aws (or equivalent Bedrock-supported module set)
- OpenSearch provider-based modules/resources

## 7) Deployment Sequence

1. Provision network and private connectivity boundaries.
2. Provision identity and access controls (IAM roles + Cognito user pool) and encryption baseline (KMS).
3. Provision persistent data plane (S3, DynamoDB, OpenSearch).
4. Provision application plane (API Gateway, Lambda ingress, agent Lambdas, Data and Image Function).
6. Provision Bedrock multi-agent orchestration and Knowledge Base integration.
7. Deploy frontend application to AWS Amplify (application code provided separately).
8. Provision observability and operational guardrails.

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
- IAM least privilege and scoped trust boundaries for every service execution role.
- Cognito user pool groups and JWT validation at the API Gateway authorizer.
- KMS encryption for data at rest and key management governance.
- Private subnet isolation for stateful services.
- Audit and threat detection coverage (GuardDuty).

## 11) Observability Requirements

- Central logs, service metrics, alarms, and dashboards.
- Distributed traces for end-to-end workflow visibility.
- Business KPIs focused on decision friction and conversion lift.

## 12) CI/CD Requirements

1. Linting and unit validation.
2. Security scanning for application code and IaC.
3. Plan/synth review gates.
4. Controlled promotion to production.
5. Amplify build triggered on source push; environment variables injected at build time.
6. Post-deploy smoke tests: Amplify URL reachable, Cognito token issued, DynamoDB seeded, Bedrock agent responds.

## 13) Validation Checklist

- Amplify app URL is reachable and returns the frontend.
- Cognito sign-in issues a valid JWT; API Gateway authorizer accepts it.
- Data and Image Function has seeded all menu categories; DynamoDB Menu Table is populated.
- Nova Canvas-generated images are accessible via CloudFront URLs.
- Voice flow completes from speech input to spoken output.
- Agent workflow returns recommendation and cart optimisation decisions.
- Knowledge retrieval is grounded and traceable.
- Security controls are active and monitored (GuardDuty, WAF, KMS).
- Observability dashboards expose latency, errors, throttles, and business KPIs.

## 14) Rollback Strategy

- Versioned runtime releases with safe rollback targets.
- Progressive delivery and rollback at service boundaries.
- Data protection and recovery posture for persistent stores.

## 15) Instructions to Coding Agent (Prompt-Only)

Use the following short, user-friendly prompt set directly with a coding agent for a 45-60 minute interview.

Prompt 1 (Build the foundation and platform): "Set up a production-ready AWS Decision Intelligence platform for UberEats 
 Step 1 — provision the Foundation Layer: Cognito user pool with AppUserGroup, API Gateway, CloudFront distribution backed by S3 for menu images, DynamoDB tables (Menu, Loyalty, Cart, Order, Chat), and the Lambda Data and Image Function that seeds the full embedded menu (entrees, main courses, soups and salads, breads, curries, sweet dishes) and calls Amazon Bedrock Nova Canvas to generate professional food photography per item, storing images in S3 for CloudFront delivery. 
 Step 2 — provision the DIPaaS Intelligence Layer on top: Bedrock multi-agent orchestration (Bedrock Agent + AgentCore + Agent Memory), Bedrock Knowledge Base backed by OpenSearch Serverless and S3, and the five agent Lambdas (intent-parser, restaurant-ranking, promotion-engine, substitution-engine, upsell-engine). Apply WAF + Shield at the edge, IAM least-privilege execution roles per service, KMS encryption, GuardDuty threat detection, and VPC private subnets. Also implement the full Alexa voice journey from speech input to spoken response."

Prompt 2 (Make it resilient and deploy the frontend): "Configure the Amplify app to deploy the application code provided by the user. Connect the Amplify app to the Cognito user pool and API Gateway endpoints provisioned in Prompt 1. Make the backend reliable at very high traffic scale with Lambda concurrency controls and idempotency on all order-impacting operations. Apply security-in-depth (IAM scoped roles, Cognito JWT validation at API Gateway, KMS at rest, GuardDuty active), and add monitoring and tracing with CloudWatch and X-Ray. Track business KPIs: decision time reduction, conversion rate, average order value, and customer satisfaction."

Prompt 3 (Ship it safely): "Add CI/CD with quality checks, security scans (IaC and application code), deployment approvals, environment promotion, and post-deploy smoke tests — including a validation that Amplify serves the frontend, Cognito issues tokens, the Data and Image Function has seeded all menu items with Nova Canvas images, DynamoDB tables contain expected data, and the Bedrock agent responds to a test intent. Add a rollback plan covering versioned Lambda releases, Amplify branch rollback, and DynamoDB point-in-time recovery. Return final deployment outputs: Amplify app URL, API Gateway endpoint, Cognito user pool and client IDs, Bedrock agent and Knowledge Base IDs, enabled security controls, scaling configuration, known risks, and a prioritised remediation list."


Deploy production-grade GenAI Decision Intelligence Platform.

## Stack

AWS:
- Cognito (user auth + groups + STS token exchange) + IAM (service execution roles)
- CloudFront (CDN for menu images)
- S3 (menu image store + data lake)
- DynamoDB (Menu, Loyalty, Cart, Order, Chat tables)
- API Gateway
- Lambda (Data and Image Function + agent Lambdas)
- Amazon Bedrock — Amazon Nova Canvas (AI food photography generation)
- Bedrock Agent + Bedrock AgentCore
- Bedrock Knowledge Base
- Agent Memory
- OpenSearch Serverless
- AWS Amplify (frontend hosting + CI/CD deployment)
- WAF + Shield + GuardDuty
- KMS
- CloudWatch + X-Ray


## Repository Structure

/app

 agents/
    intent_agent.py
    restaurant_agent.py
    negotiation_agent.py
    substitution_agent.py
    upsell_agent.py

 services/

    api_gateway/
    lambda/
       data_image_function/   # seeds menu + calls Nova Canvas for food photography
    ingestion/

 frontend/                    # application code provided by user; deployed to AWS Amplify

 infra/

    terraform/

 tests/


## Python Dependencies

boto3
aws-lambda-powertools
langchain
bedrock-agent-runtime
opensearch-py
sqlalchemy
fastapi
pydantic


## Deploy Steps


1. Create VPC and network boundaries

2. Create IAM roles (least privilege per service)

3. Provision Cognito User Pool
   - Create AppUserGroup with /appuser path
   - Configure STS token vending and JWT issuance

4. Provision S3 Buckets
   - Standard image bucket (menu images origin)
   - Data lake bucket (menu/order data)

5. Provision CloudFront Distribution
   - Origin: S3 image bucket
   - WAF association for edge protection

6. Deploy DynamoDB Tables
   - Menu Table
   - Loyalty Table
   - Cart Table
   - Order Table
   - Chat Table

7. Deploy Lambda Data and Image Function
   - Embed complete menu catalogue (entrees, main courses, soups and salads, breads, curries, sweet dishes)
   - Invoke Amazon Bedrock Nova Canvas per menu item to generate professional food photography
   - Store generated images in S3 for CloudFront delivery
   - Seed DynamoDB Menu Table with items and image URLs

8. Deploy OpenSearch

9. Create Bedrock Knowledge Base

10. Load restaurant data from S3

11. Create Aurora schema (if relational joins are required for analytics)

Tables:

restaurants
menus
inventory
orders
promotions
customers


12. Deploy agent Lambdas

Functions:

intent-parser
restaurant-ranking
promotion-engine
substitution-engine
upsell-engine


13. Deploy API Gateway

Endpoints:

POST /conversation
POST /recommend
POST /order


15. Deploy frontend to AWS Amplify
   - Connect Amplify app to the Git repository (application code provided by user)
   - Set environment variables: Cognito User Pool ID, Cognito Client ID, API Gateway URL, CloudFront domain
   - Trigger build and deployment
   - Validate Amplify app URL resolves and Cognito sign-in flow works end-to-end


16. Enable monitoring

CloudWatch:
- latency
- token usage
- conversion rate
- abandonment


## Success Metrics

Track:

Decision Time Reduction

Conversion Rate

Average Order Value

Merchant Revenue Lift

Customer Satisfaction
