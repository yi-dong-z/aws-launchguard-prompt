# DoraHacks Copy-Paste Submission Guide

Use this file when filling the DoraHacks BUIDL form for AWS Prompt the Planet Challenge.

## Project Name

AWS LaunchGuard

## Project Tagline

Cost and security guardrails for startup SaaS MVPs on AWS.

## Short Description

AWS LaunchGuard is a production-ready prompt package that helps early-stage founders generate a low-cost, security-default, serverless-first AWS launch plan for a SaaS API MVP.

## Full Description

AWS LaunchGuard is designed for founders and small startup teams who need to ship an AWS-backed SaaS API MVP without a dedicated cloud architect. Generic AI-generated infrastructure advice can be too broad, too expensive, or missing safety steps. AWS LaunchGuard fixes that by giving the LLM a strict workflow: ask the right intake questions first, then generate a practical AWS launch plan with cost and security guardrails built in.

The prompt defaults to a USD 50/month budget, `us-east-1`, dev/staging usage, low MVP traffic, Terraform, and non-regulated customer data. It prefers serverless AWS services such as Lambda, API Gateway HTTP API, DynamoDB on-demand, S3 with Block Public Access, EventBridge, CloudWatch Logs, AWS Budgets, and IAM least privilege.

It explicitly avoids costly or operationally heavy defaults such as NAT Gateway, EKS, OpenSearch, GPU instances, long-running EC2, and always-on RDS unless the user confirms the tradeoff.

The generated output follows a fixed 10-section structure: assumptions, recommended architecture, cost guardrails, security guardrails, Terraform starter kit, deployment steps, validation commands, troubleshooting, rollback and destroy, and AWS Well-Architected alignment.

This submission includes the universal prompt, a Kiro steering version, a SaaS API MVP example input and output, offline validation scenarios, an optional USD 5 smoke test path, and a 60-second demo script.

## Complete Prompt

Paste the full contents of `prompt.md` into the Details section if the form asks for the complete prompt.

## Context & Documentation

Prerequisites:

- A capable LLM such as Claude, GPT, Amazon Bedrock, Kiro, or another agentic coding assistant.
- Basic AWS account awareness if the user chooses to run the optional smoke test.
- Terraform knowledge is helpful but not required for understanding the output.
- No real AWS deployment is required for validating the prompt.

Use case:

AWS LaunchGuard helps early-stage founders and small startup teams safely plan a SaaS API MVP on AWS. It is especially useful when the team needs a practical serverless architecture, Terraform starter kit, cost controls, and security defaults before launching.

Expected outcome:

After using the prompt, developers receive a clear AWS launch plan with assumptions, serverless architecture, budget alerts, security guardrails, Terraform starter code, deployment steps, validation commands, troubleshooting, cleanup, and AWS Well-Architected alignment.

Troubleshooting tips:

- If the user gives too little information, the prompt asks up to 8 focused intake questions instead of generating unsafe infrastructure.
- If the user requests high-cost resources such as EKS, RDS, OpenSearch, NAT Gateway, GPU, or EC2, the prompt explains the cost and complexity risk and suggests a serverless alternative.
- If Terraform output is produced, the user should run `terraform validate` and `terraform plan` before applying anything.
- The prompt always reminds users that AWS charges depend on region, account status, traffic, service usage, and cleanup.

## AWS Services Used

- AWS Lambda
- Amazon API Gateway HTTP API
- Amazon DynamoDB on-demand
- Amazon S3 with Block Public Access
- Amazon EventBridge
- Amazon CloudWatch Logs
- AWS Budgets
- AWS IAM
- Optional CloudFront, Parameter Store, or Secrets Manager when needed

## AWS Best Practices Alignment

AWS LaunchGuard aligns with AWS Well-Architected by making launch safety explicit:

- Operational Excellence: repeatable Terraform, validation commands, troubleshooting, and rollback steps.
- Security: S3 Block Public Access, least-privilege IAM, no hardcoded secrets, restrictive CORS, and private data defaults.
- Reliability: serverless managed services reduce operational burden for small teams.
- Performance Efficiency: Lambda, API Gateway, and DynamoDB scale with MVP demand without upfront capacity planning.
- Cost Optimization: budget alerts, short log retention, DynamoDB on-demand, and avoidance of high-cost defaults.
- Sustainability: serverless scaling reduces idle infrastructure for low-traffic workloads.

## Suggested Tags

Prompt Engineering, AWS, Cloud Architecture, Developer Tools, Generative AI, Kiro, Terraform, Serverless, Cost Optimization, Security

## GitHub Repository URL

Create a public GitHub repo named:

```text
aws-launchguard-prompt
```

Then paste the public repo URL here:

```text
https://github.com/YOUR_USERNAME/aws-launchguard-prompt
```

## Demo Video URL

After recording the 60-second demo, paste the video URL here.

## Demo Script Source

Use `demo-script.md`.

## Track / Technology

If the form asks for a track or technology, choose the AWS Prompt the Planet / Prompt Engineering / Prompt Library option.

## Team

Solo builder is acceptable unless you want to add teammates.
