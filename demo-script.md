# 60-Second Demo Script

## Title

AWS LaunchGuard: Cost and Security Guardrails for Startup SaaS MVPs

## Script

Hi, this is AWS LaunchGuard, a prompt package for early-stage founders who need to launch a SaaS API MVP on AWS without accidentally creating an expensive or insecure cloud setup.

The problem is simple: startup teams move fast, but AWS decisions like databases, networking, IAM, logs, S3 access, and billing alerts can create risk before the product even has users.

AWS LaunchGuard turns that into a repeatable LLM workflow.

First, the prompt checks whether it has enough context. If the founder only gives a vague idea, it asks focused questions about budget, region, traffic, data sensitivity, required features, environment, and infrastructure preference.

Once the brief is complete, it generates a serverless-first launch plan. The default path uses Lambda, API Gateway HTTP API, DynamoDB on-demand, S3 with Block Public Access, EventBridge for lightweight jobs, CloudWatch log retention, IAM least privilege, and AWS Budgets.

It also avoids expensive defaults like NAT Gateway, EKS, OpenSearch, GPU instances, long-running EC2, and always-on RDS unless the user explicitly accepts the tradeoff.

The output includes assumptions, architecture, cost guardrails, security guardrails, Terraform starter code, deployment steps, validation commands, troubleshooting, rollback, destroy instructions, and Well-Architected alignment.

The primary submission is offline-verifiable, with an optional smoke test capped by a USD 5 target. AWS LaunchGuard helps founders treat cost and security as launch requirements, not cleanup tasks.
