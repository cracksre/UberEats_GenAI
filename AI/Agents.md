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

- The new Decision Intelligence system: the same Alexa channel plus Bedrock Agent (Amazon Nova Sonic model ), which shall take care of the various tasks, namely, intent/ranking/negotiation/substitution/promotion and trigger API gateway to interact with DynamoDB table. The Amazon Nova canvas model will be triggered by the Almda function to generate picture of menu items chosen and stored in S# bucket using knowledge retrieval and distribution(KB/CDN), event backbone, and DynamoDB to keep the cost low.

# agent.md - Coding Agent Deployment Playbook (UberEats DIPaaS)

This runbook gives a coding agent a deterministic sequence to provision and deploy the Decision Intelligence Platform as a Service (DIPaaS) on AWS.

## 1) Objective

Deploy a multi-agent architecture that:

1. Understands user intent from chat and voice.
2. Finds best restaurant outcomes (value + ETA + wait-time + preference fit).
3. Negotiates and optimizes cart outcomes.
4. Handles substitutions for 86'd items and upsell logic.
5. Applies restaurant promotions while maximizing customer savings and business profit.

## Decision Intelligence Workflow

1. Access the digital menu via voice commands using Alexa voice assistant to the application hosted on Amplify.
2. Amazon Cognito manages temporary AWS credentials with scoped IAM permissions and secured access to backend service through token based authorization.
3. The Digital menu uses the AWS SDK Biderictinal streaming API to establish a SigV4 signed Websocket session with Amazon Bedrock, a fully managed service with built in security, privacy and responsible AI.
4. The app passes tool definitions and a system prompt to Amazon Nova2 Sonic enabling the model to orchestrate tool calls and streaming audio.
5. Amazon Nova 2 Sonic processes your streaming audio and invokes tools defined at the application level.
6. The Applications tool router captures model requests with parameters to manipulate the UI and make API calls.
7. You initiate the voice input based on Alexa voice and Alexa skills as a trigger through the Bedrock agent, the API gateway. 
8. The API gateway routes tool requests to DynamoDB tables for menu, cart, order, loyalty and chat data.
9. Amazon CloudFront delivers AI generated menu images stored in Amazon S3 using Amazon Opensearch , protected by Amazon WAF, that filters malicious traffic and prevents malicious attacks.
10. Amazon Lambda triggers the Amazon Nova Canvas AI model to generate the image and stores the index in S3 and updates the image reference in the DynamoDB menu table (Happens during initial deployment), OR during addition of a new dish in the menu.
11. Amazon KMS encrypts the Dynamod DB table.
12. Amazon SQS processes DLQ for failed Amazon Lamda invocations for retries.
13. Amazon X-Ray captures Agents metrics and logs
14. Amazon Cloudwatch collects logs and Cloudtrail collects traces.



## 2) Service Options and Why Chosen

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

## 6) Terraform Module (main.tf)

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

## 8) Scale and Reliability Requirements

- Support millions of requests through async fan-out and buffering.
- Enforce idempotency for all order-impacting operations.
- Apply retries and dead-letter routing for failed events.
- Configure autoscaling and concurrency controls for ingress.

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


# Backend (to be provisioned — not yet scaffolded in this repo)
# agents/
#    intent_agent.py
#    restaurant_agent.py
#    negotiation_agent.py
#    substitution_agent.py
#    upsell_agent.py
# services/
#    api_gateway/
#    lambda/
#       data_image_function/   # seeds menu + calls Nova Canvas for food photography
#    ingestion/
# infra/
#    terraform/
# tests/
#
# Python dependencies (required when backend is scaffolded):
#   boto3, botocore, pydantic, fastapi, mangum, opensearch-py, uvicorn
#   aws-lambda-powertools, langchain, bedrock-agent-runtime, sqlalchemy


## Deploy Steps (instructions to agent)


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


12. Deploy agent Lambda

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
