# Validation Plan

AWS LaunchGuard is designed to be validated offline. The prompt package should prove that it can guide an LLM toward practical, cost-aware, security-default AWS output without requiring real AWS deployment.

## Offline Scenario 1: Missing Information

Input:

```text
I have a startup idea and need AWS infrastructure. Please generate Terraform.
```

Expected behavior:

- The LLM should not generate Terraform immediately.
- It should ask up to 8 intake questions.
- The questions should focus on product purpose, region, budget, traffic, data sensitivity, required features, environment, and IaC preference.
- It should mention safe defaults only as defaults, not as confirmed facts.

Pass criteria:

- No final architecture is generated.
- No Terraform is generated.
- The questions would materially affect cost, security, or architecture.

## Offline Scenario 2: Standard SaaS API MVP

Input:

- Use `examples/saas-api-input.md`.

Expected behavior:

- Output follows the required 10-section structure.
- Default or supplied budget is `USD 50/month`.
- Architecture is serverless-first.
- Terraform includes API Gateway HTTP API, Lambda, DynamoDB on-demand, S3 with Block Public Access, EventBridge for the requested background job, CloudWatch log retention, IAM, and AWS Budgets.
- Terraform does not include NAT Gateway, EKS, OpenSearch, GPU, long-running EC2, or RDS.
- Output includes deployment, validation, troubleshooting, rollback, and destroy steps.

Pass criteria:

- Cost and security guardrails appear before deployment instructions.
- Cleanup is explicit.
- Actual-cost warning is included.

## Offline Scenario 3: High-Cost Request

Input:

```text
Build my MVP on EKS with RDS, OpenSearch, and a NAT Gateway. I expect 200 beta users and have a USD 50 monthly budget.
```

Expected behavior:

- The LLM should not blindly generate EKS/RDS/OpenSearch/NAT Gateway Terraform.
- It should explain why the requested services are high-cost or operationally heavy for this budget and traffic level.
- It should propose a serverless alternative.
- It should ask for explicit confirmation before including the high-cost services.

Pass criteria:

- High-cost services are flagged.
- A serverless alternative is offered.
- The user must explicitly accept the cost/complexity tradeoff before those services appear in the architecture.

## Example Output Checklist

Use this checklist against `examples/saas-api-output.md` or a fresh LLM response:

- [ ] Includes all 10 required sections in order.
- [ ] Uses `us-east-1` unless another region is specified.
- [ ] Uses `USD 50/month` as the default budget.
- [ ] Includes budget alert thresholds at 50%, 80%, and 100%.
- [ ] Uses DynamoDB on-demand or gives a low-capacity reason.
- [ ] Blocks public access on S3.
- [ ] Does not expose uploaded objects directly to the public internet.
- [ ] Uses restrictive CORS, not wildcard CORS.
- [ ] Avoids hardcoded secrets.
- [ ] Includes IAM least-privilege guidance.
- [ ] Includes CloudWatch log retention.
- [ ] Includes EventBridge or a clearly marked equivalent when background jobs are requested.
- [ ] Includes `terraform validate`, `terraform plan`, and review-before-apply guidance.
- [ ] Includes `terraform destroy`.
- [ ] Warns that AWS charges vary by account, region, traffic, and cleanup.

## Optional AWS Smoke Test

The primary submission does not require AWS deployment. If you choose to run a smoke test, use a dedicated sandbox AWS account or a clearly isolated dev environment.

Target spend ceiling: `USD 5`.

Before testing:

1. Confirm which AWS account and region you are using.
2. Create or confirm a billing alert or AWS Budget.
3. Use a real email for budget notifications and confirm it if AWS asks.
4. Run `terraform plan` and inspect the resource list.
5. Stop if the plan includes NAT Gateway, EKS, OpenSearch, GPU, EC2, or RDS.

Minimal smoke test:

```bash
aws sts get-caller-identity
terraform init
terraform validate
terraform plan \
  -var="budget_alert_email=founder@example.com" \
  -var="allowed_cors_origin=https://app.example.com"
```

Only apply after confirming the plan:

```bash
terraform apply \
  -var="budget_alert_email=founder@example.com" \
  -var="allowed_cors_origin=https://app.example.com"
curl "$(terraform output -raw api_endpoint)/health"
terraform destroy \
  -var="budget_alert_email=founder@example.com" \
  -var="allowed_cors_origin=https://app.example.com"
```

After testing:

```bash
aws ce get-cost-and-usage \
  --time-period Start=YYYY-MM-DD,End=YYYY-MM-DD \
  --granularity MONTHLY \
  --metrics UnblendedCost
```

Important: A `USD 5` target is a guardrail, not a guarantee. Actual AWS charges depend on account status, region, service pricing, traffic, logs, data transfer, and cleanup.

## AWS Best-Practice References

- AWS Budgets pricing: https://aws.amazon.com/aws-cost-management/aws-budgets/pricing/
- AWS Lambda pricing: https://aws.amazon.com/lambda/pricing/
- AWS Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html
- Amazon S3 Block Public Access: https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html
