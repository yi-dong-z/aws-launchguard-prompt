# AWS LaunchGuard

Cost and security guardrails for startup SaaS MVPs.

AWS LaunchGuard is a prompt package for early-stage founders and small product teams who need to ship a SaaS API MVP on AWS without a dedicated cloud architect. It helps an LLM ask the right intake questions first, then generate a low-cost, serverless-first AWS launch plan with Terraform starter code, budget alerts, security defaults, validation commands, troubleshooting guidance, and cleanup steps.

This is designed for the AWS Prompt the Planet Challenge. The primary deliverable is the prompt itself, not a deployed application.

GitHub repository: https://github.com/yi-dong-z/aws-launchguard-prompt

## Problem

Early startup teams often deploy too quickly and discover cloud risks late:

- Costs creep in through always-on services, NAT gateways, databases, log retention, and forgotten resources.
- S3 buckets, IAM roles, APIs, and secrets are easy to misconfigure under deadline pressure.
- Architecture suggestions from generic AI assistants can be too broad, too expensive, or missing deletion steps.
- Founders need a practical path for a small MVP before they need enterprise infrastructure.

AWS LaunchGuard turns those concerns into a repeatable prompt workflow.

## Target User

The prompt is optimized for:

- Early-stage startup founders.
- Hackathon teams turning a prototype into a real MVP.
- Solo developers who are comfortable with code but not with AWS architecture decisions.
- Product engineers who want serverless defaults, cost boundaries, and security checks before deployment.

The default example is a SaaS API MVP with low traffic, non-regulated customer data, and a target budget of USD 50 per month.

## What The Prompt Produces

When the user provides a startup or product description, AWS LaunchGuard produces:

- Clear assumptions and missing-information questions.
- A recommended serverless AWS architecture.
- Cost guardrails and budget alert recommendations.
- Security guardrails for IAM, S3, API access, logs, secrets, and encryption.
- Terraform starter code that avoids expensive default resources.
- Deployment and validation commands.
- Troubleshooting guidance for common AWS issues.
- Rollback and destroy instructions.
- AWS Well-Architected alignment notes.

## Design Principles

- Ask before generating when critical information is missing.
- Prefer serverless services for MVPs: Lambda, API Gateway HTTP API, DynamoDB, S3, EventBridge, CloudFront, CloudWatch, AWS Budgets, and IAM.
- Avoid high-cost defaults such as NAT Gateway, EKS, OpenSearch, GPU instances, long-running EC2, and RDS unless the user explicitly accepts the tradeoff.
- Make cleanup a first-class part of the output.
- Treat cost and security as launch requirements, not post-launch improvements.

## Package Contents

- `prompt.md` - Universal copy-paste prompt for Claude, GPT, Amazon Bedrock, Kiro, or another capable LLM.
- `.kiro/steering/aws-launchguard.md` - Kiro steering version for project-level guidance.
- `kiro-demo.md` - Kiro usage and validation notes.
- `examples/saas-api-input.md` - Example user input for a SaaS API MVP.
- `examples/saas-api-output.md` - Example response generated from the prompt.
- `examples/chatgpt-real-run.md` - Real ChatGPT web run notes for the TaskPulse example.
- `validation.md` - Offline validation scenarios and optional AWS smoke test guide.
- `demo-script.md` - 60-second demo video script.
- `submission-summary.md` - DoraHacks submission copy.

## How To Use

1. Copy the full prompt from `prompt.md`.
2. Paste it into a capable LLM.
3. Provide a concise startup or product description.
4. Answer the intake questions if the LLM asks for missing details.
5. Review the generated architecture, Terraform, budget settings, and security guardrails.
6. Use `validation.md` to check whether the output behaves as intended.

For Kiro, add `.kiro/steering/aws-launchguard.md` to the project and use it as persistent project guidance before asking Kiro to generate AWS deployment artifacts.

## Safety Note

The primary submission is offline-verifiable and does not require creating AWS resources. Any optional AWS smoke test should start with a budget alert and end with resource destruction. Actual AWS charges depend on account status, region, traffic, service configuration, and whether resources are cleaned up.

## Sources Used For Guardrail Design

- AWS Budgets pricing: https://aws.amazon.com/aws-cost-management/aws-budgets/pricing/
- AWS Lambda pricing and free tier details: https://aws.amazon.com/lambda/pricing/
- AWS Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html
- Amazon S3 Block Public Access: https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html
