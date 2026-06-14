UberEats DIPaaS Architecture Flow Guide

Diagram file:
- docs/architecture/ubereats-dipaas-architecture.drawio

Purpose:
This architecture shows an end-to-end, production-grade Decision Intelligence Platform as a Service (DIPaaS) for Uber Eats to reduce decision friction and improve conversion.

Primary business outcome:
- Help users decide faster with a concierge workflow.
- Increase conversion and average order value.
- Reduce abandonment, churn, and refunds.

End-to-end flow (numbered path in diagram):
1) User speaks through Alexa Skill.
2) Amazon Transcribe converts speech to text.
3) API Gateway receives the request through edge controls.
4) Lambda Ingress sends request to Amazon Bedrock Agents orchestrator.
5) Intent Agent interprets customer intent.
6) Ranking Agent identifies best restaurant choices based on constraints.
7) Negotiation Agent optimizes pricing/offer scenarios.
8) Substitution/Upsell Agent handles 86'd items and better alternatives.
9) Promotions Agent applies eligible offers.
10) Orchestrator queries Amazon Bedrock Knowledge Bases.
11) Knowledge Bases retrieve indexed context from Amazon OpenSearch Service.
12) Knowledge Bases retrieve source context from Amazon S3.
13) Decision and state updates persist in Aurora PostgreSQL.
14) Merchant/POS integrations are called for menu, stock, and ETA confirmation.
15) Response text is converted to speech by Amazon Polly.
16) Spoken response is returned to Alexa.

High-scale event path:
- Lambda Ingress emits domain events to Amazon EventBridge.
- EventBridge fans out to Kinesis and SQS.
- SQS can route failed messages to DLQ.
- Worker Lambdas process asynchronous enrichment and persistence.

Security-in-depth:
- AWS Shield + AWS WAF at edge.
- AWS IAM for least privilege.
- AWS KMS for encryption keys.
- Secrets Manager for secrets.
- VPC for private network boundaries.
- CloudTrail/Config/GuardDuty for audit and threat detection.

Observability:
- CloudWatch metrics and alarms.
- CloudWatch Logs for runtime logging.
- AWS X-Ray for distributed tracing.
- KPI board for executive visibility: decision friction, conversion, order value, churn, refunds.

What Bedrock service maps to orchestrator?
- The "Bedrock Multi-Agent Orchestrator" is represented by Amazon Bedrock Agents.

Editing/export instructions:
1) Open the drawio file in app.diagrams.net.
2) Enable AWS icon library if needed.
3) Edit labels and connectors.
4) Export as PNG/SVG/PDF for presentation.
