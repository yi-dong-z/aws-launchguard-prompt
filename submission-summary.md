# DoraHacks Submission Summary

## Project Name

AWS LaunchGuard: Cost and Security Guardrails for Startup SaaS MVPs

## Short Description

AWS LaunchGuard is a prompt package that helps early-stage founders generate a low-cost, security-default, serverless-first AWS launch plan for a SaaS API MVP.

## Full Description

AWS LaunchGuard is designed for founders and small startup teams who need to ship an AWS-backed SaaS API MVP without a dedicated cloud architect. Generic AI-generated infrastructure advice can be too broad, too expensive, or missing safety steps. AWS LaunchGuard fixes that by giving the LLM a strict workflow: ask the right intake questions first, then generate a practical AWS launch plan with cost and security guardrails built in.

The prompt defaults to a USD 50/month budget, `us-east-1`, dev/staging usage, low MVP traffic, Terraform, and non-regulated customer data. It prefers serverless AWS services such as Lambda, API Gateway HTTP API, DynamoDB on-demand, S3 with Block Public Access, EventBridge, CloudWatch Logs, AWS Budgets, and IAM least privilege.

It also explicitly avoids costly or operationally heavy defaults such as NAT Gateway, EKS, OpenSearch, GPU instances, long-running EC2, and always-on RDS unless the user confirms the tradeoff.

The generated output follows a fixed 10-section structure: assumptions, recommended architecture, cost guardrails, security guardrails, Terraform starter kit, deployment steps, validation commands, troubleshooting, rollback and destroy, and AWS Well-Architected alignment.

This submission includes the universal prompt, a Kiro steering version, a SaaS API MVP example input and output, offline validation scenarios, an optional USD 5 smoke test path, and a 60-second demo script.

## Why It Matters

For startup teams, the riskiest AWS mistakes often happen before scale: forgotten resources, public buckets, broad IAM policies, missing billing alerts, unnecessary always-on services, and no cleanup plan. AWS LaunchGuard makes those concerns part of the first architecture response instead of an afterthought.

## What Is Included

- Universal copy-paste prompt.
- Kiro steering prompt.
- SaaS API MVP example input.
- Example generated output.
- Offline validation plan.
- Optional AWS smoke test guide.
- 60-second demo script.

## Intended Users

- Early-stage startup founders.
- Hackathon teams converting prototypes into real MVPs.
- Solo developers launching on AWS for the first time.
- Product engineers who want safer AWS defaults before deployment.

## Offline Validation

The prompt can be tested without creating AWS resources:

1. Give it a vague startup idea and verify that it asks intake questions.
2. Give it the included SaaS API MVP input and verify that it generates the required 10-section output.
3. Ask for EKS, RDS, OpenSearch, and NAT Gateway under a USD 50 budget and verify that it warns about cost and proposes a serverless alternative.

## Optional Smoke Test

The package includes an optional AWS smoke test path with a USD 5 target spend ceiling. The smoke test requires budget alerts before deployment, review of `terraform plan`, a basic API test, and immediate `terraform destroy`.

Actual AWS charges depend on account status, region, traffic, service pricing, logs, data transfer, and cleanup.
