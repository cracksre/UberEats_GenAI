# Uber Eats DIPaaS Deployment Agent

#Business Case:

#Task


## Objective

Deploy production-grade GenAI Decision Intelligence Platform.

## Stack

AWS:
- Bedrock
- Bedrock Agents
- Bedrock Knowledge Base
- OpenSearch Serverless
- Aurora PostgreSQL
- Lambda
- API Gateway
- EventBridge
- S3
- IAM
- KMS
- CloudWatch


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
    ingestion/

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


1. Create VPC

2. Create IAM roles

3. Deploy OpenSearch

4. Create Bedrock Knowledge Base

5. Load restaurant data from S3

6. Create Aurora schema


Tables:

restaurants
menus
inventory
orders
promotions
customers


7. Deploy Lambda functions


Functions:

intent-parser

restaurant-ranking

promotion-engine

substitution-engine

upsell-engine


8. Create EventBridge rules


Events:

MENU_UPDATED

ITEM_OUT_OF_STOCK

ORDER_CREATED


9. Deploy API Gateway


Endpoints:

POST /conversation

POST /recommend

POST /order


10. Enable monitoring

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
