# ChatGPT Real LLM Run

This file records a real ChatGPT web run of the AWS LaunchGuard prompt package for the TaskPulse SaaS API MVP example.

## Run Metadata

- Date: 2026-06-05
- Surface: ChatGPT web
- Conversation title shown by ChatGPT: `AWS MVP LaunchGuard Plan`
- Demo input: `examples/saas-api-input.md`
- Note: The run was performed through the user's logged-in ChatGPT browser session. Clipboard export was not used in the repository because direct system clipboard reads were blocked by local safety policy, so this file records the visible generated output and checked sections.

## Demo Prompt Sent

The run used the AWS LaunchGuard behavior rules from `prompt.md`, including:

- Serverless-first AWS MVP architecture.
- Defaults of `us-east-1`, `USD 50/month`, `dev/staging`, low MVP traffic, non-regulated customer data, and Terraform.
- Intake questions when critical information is missing.
- Avoiding NAT Gateway, EKS, OpenSearch, GPU, long-running EC2, always-on RDS, Multi-AZ dev/staging databases, and unnecessary VPC-attached Lambda.
- Required 10-section output:
  1. Assumptions
  2. Recommended Architecture
  3. Cost Guardrails
  4. Security Guardrails
  5. Terraform Starter Kit
  6. Deployment Steps
  7. Validation Commands
  8. Troubleshooting
  9. Rollback and Destroy
  10. Well-Architected Alignment

The sample product was TaskPulse, a SaaS API MVP for weekly team status updates with API, database, CSV uploads, authentication placeholder, daily background job, and basic logs.

## Observed ChatGPT Output

ChatGPT generated the requested 10-section AWS LaunchGuard output.

### 1. Assumptions

It identified:

- Region: `us-east-1`
- Environment: `dev/staging`
- Budget ceiling: `USD 50/month`
- Traffic: fewer than 1,000 users and fewer than 100,000 API requests/month
- Data: non-regulated customer data, not public
- IaC: Terraform
- Auth: placeholder now, with room for Cognito later
- Launch priority: low-cost, secure-by-default, serverless-first MVP

It also included the cost safety note that AWS charges depend on region, account status, traffic, service usage, and cleanup.

### 2. Recommended Architecture

The generated architecture used:

- Amazon API Gateway HTTP API for public HTTPS API
- AWS Lambda for API handlers
- Amazon DynamoDB on-demand for teams, users, check-ins, and summaries
- Amazon S3 for CSV imports with Block Public Access
- EventBridge Scheduler for the daily missing-check-in job
- CloudWatch Logs with explicit retention
- SSM Parameter Store for non-sensitive config
- Secrets Manager only if true secrets are needed
- AWS Budgets with 50%, 80%, and 100% alerts

It explicitly avoided NAT Gateway, EKS, OpenSearch, always-on EC2, always-on RDS, and VPC-attached Lambda unless required.

### 3. Cost Guardrails

The run included:

- AWS Budget set to `USD 50/month`
- Alerts at `USD 25`, `USD 40`, and `USD 50`
- DynamoDB `PAY_PER_REQUEST`
- Lambda without VPC by default
- API Gateway HTTP API instead of REST API
- CloudWatch Logs retention of 14 days
- S3 lifecycle guidance for temporary uploads
- A Secrets Manager cost note

### 4. Security Guardrails

The run included:

- S3 Block Public Access
- No hardcoded secrets
- Least-privilege IAM
- Restrictive CORS
- DynamoDB encryption at rest by default
- S3 server-side encryption
- Private upload bucket and presigned URL guidance
- Authentication placeholder warning for privileged actions
- Explicit CloudWatch log retention

### 5. Terraform Starter Kit

ChatGPT generated a compact HCL starter kit. The visible output confirmed Terraform resources and snippets for:

- AWS provider
- Project/environment/budget/email/CORS variables
- API Gateway HTTP API
- Lambda package placeholder
- DynamoDB on-demand table
- S3 upload bucket with public access blocked
- CloudWatch log group
- EventBridge schedule
- AWS Budget notifications including 100% threshold
- `data "aws_caller_identity" "current" {}`
- `output "api_url"` using the API Gateway HTTP API endpoint

The generated Terraform did not include NAT Gateway, EKS, OpenSearch, GPU, long-running EC2, or RDS.

### 6. Deployment Steps

ChatGPT generated:

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

It also instructed the user to package Lambda code into `dist/api.zip`, replace placeholder email/CORS values, store runtime config in Parameter Store, and add Cognito later when real authentication is needed.

### 7. Validation Commands

The run included commands such as:

```bash
terraform output api_url
curl "$(terraform output -raw api_url)/health"

aws dynamodb describe-table \
  --table-name taskpulse-staging

aws s3api get-public-access-block \
  --bucket taskpulse-staging-uploads-ACCOUNT_ID

aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/taskpulse-staging
```

### 8. Troubleshooting

The output covered:

- API returns 500: check Lambda logs in CloudWatch
- CORS failure: confirm frontend origin exactly matches Terraform CORS origin
- S3 upload denied: check Lambda IAM policy and bucket name
- DynamoDB access denied: verify table ARN in Lambda IAM policy
- Unexpected AWS bill: check Cost Explorer and confirm no NAT Gateway, EC2, RDS, OpenSearch, or unused test resources were created

### 9. Rollback and Destroy

The run included rollback and destroy commands:

```bash
terraform plan
terraform apply
terraform destroy
```

It also warned to export needed DynamoDB or S3 data before destroy and verify S3 buckets, log groups, and scheduled jobs are gone afterward.

### 10. Well-Architected Alignment

The generated mapping covered:

- Cost Optimization: serverless-first, DynamoDB on-demand, HTTP API, budget alerts, no NAT Gateway
- Security: private S3, least-privilege IAM, no hardcoded secrets, restrictive CORS
- Reliability: managed AWS services, simple failure domains, EventBridge for scheduled jobs
- Operational Excellence: Terraform IaC, CloudWatch logs, explicit validation commands
- Performance Efficiency: Lambda and DynamoDB scale automatically for MVP traffic
- Sustainability: pay-per-use services and automatic cleanup reduce idle waste

## Demo Verdict

The ChatGPT run successfully demonstrated the intended AWS LaunchGuard behavior:

- It used the complete 10-section output structure.
- It stayed serverless-first and avoided expensive default services.
- It included budget, security, Terraform, validation, troubleshooting, rollback, and cleanup guidance.
- It preserved the `USD 50/month` budget target and emphasized that real AWS costs depend on account, region, usage, and cleanup.
