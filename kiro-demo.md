# Kiro Demo Notes

AWS LaunchGuard includes a Kiro steering file at:

```text
.kiro/steering/aws-launchguard.md
```

## How To Test In Kiro

1. Open this repository in Kiro.
2. Confirm the steering file is available under `.kiro/steering/`.
3. Ask Kiro to generate an AWS launch plan for the sample product in `examples/saas-api-input.md`.
4. Check that Kiro follows the LaunchGuard rules:
   - It asks intake questions if required information is missing.
   - It prefers Lambda, API Gateway HTTP API, DynamoDB, S3, EventBridge, CloudWatch, AWS Budgets, and IAM.
   - It avoids NAT Gateway, EKS, OpenSearch, GPU, long-running EC2, and RDS by default.
   - It includes cost guardrails, security guardrails, Terraform starter code, validation commands, troubleshooting, rollback, and destroy steps.

## Expected Result

Kiro should behave like a cost-aware AWS startup solutions architect. For the TaskPulse SaaS API MVP example, it should recommend a low-cost serverless architecture with Terraform and explicit cleanup guidance rather than a broad enterprise architecture.

## Evidence To Capture

For a final video or screenshot, show:

- The `.kiro/steering/aws-launchguard.md` file.
- The sample input from `examples/saas-api-input.md`.
- Kiro's generated architecture and Terraform sections.
- The cost and security guardrail sections.
