# Architecture Transition: From Legacy Customer Service to Decision Intelligence

The designed solution should now include a second architecture diagram that shows:

- The old customer service system: Alexa voice input → speech-to-text → API gateway → request processing → static response generation → text-to-speech output.

- The new Decision Intelligence system: the same Alexa channel plus Bedrock Agent (Amazon Nova Sonic model ), which shall take care of the various tasks, namely, intent/ranking/negotiation/substitution/promotion and trigger API gateway to interact with DynamoDB table. The Amazon Nova canvas model will be triggered by the Almda function to generate picture of menu items chosen and stored in S# bucket using knowledge retrieval and distribution(KB/CDN), event backbone, and DynamoDB to keep the cost low.


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
