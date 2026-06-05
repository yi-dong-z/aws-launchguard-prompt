# Example Output: SaaS API MVP

This example shows the intended behavior of the AWS LaunchGuard prompt for the TaskPulse SaaS API MVP input. It is an illustrative output for offline validation and review.

## 1. Assumptions

- Product: TaskPulse, a SaaS API MVP for weekly team check-ins and project health summaries.
- AWS region: `us-east-1`.
- Monthly budget target: `USD 50`.
- Environment: `dev/staging`.
- Expected traffic: fewer than 1,000 users and fewer than 100,000 API requests in the first month.
- Data sensitivity: non-regulated customer data, but private by default.
- IaC: Terraform.
- Authentication is a placeholder for v1. A real production launch should add Cognito, a custom authorizer, or another authenticated identity provider before storing sensitive data.

## 2. Recommended Architecture

Use a small serverless architecture:

- Users call an API Gateway HTTP API over HTTPS.
- API Gateway routes requests to a Lambda function.
- Lambda stores teams, users, check-ins, and summaries in DynamoDB.
- CSV uploads go to a private S3 bucket with Block Public Access enabled.
- A scheduled EventBridge rule triggers the same Lambda or a separate worker Lambda once per day to detect missing check-ins.
- CloudWatch Logs stores application logs with a short retention period.
- AWS Budgets alerts the founder before the project exceeds the budget.

Intentionally left out:

- No NAT Gateway, because the default Lambda does not need private subnet egress.
- No EKS, because the MVP does not need Kubernetes.
- No OpenSearch, because search is not a v1 requirement.
- No RDS, because DynamoDB on-demand is simpler and avoids always-on database cost for low traffic.
- No long-running EC2 instances.

## 3. Cost Guardrails

- Budget target: `USD 50/month`.
- Create budget alerts at 50%, 80%, and 100% of the monthly target.
- Use DynamoDB on-demand billing for unpredictable low traffic.
- Set CloudWatch log retention to 14 days for dev/staging.
- Keep uploaded CSV files private and add a lifecycle rule later if imports become large or temporary.
- Keep Lambda outside a VPC unless a private network dependency is introduced.
- Review `terraform plan` for any unexpected resource containing names like `nat_gateway`, `eks`, `opensearch`, `rds`, `ec2`, or `elasticache`.
- Actual AWS costs depend on account status, region, traffic, data transfer, service configuration, and whether resources are destroyed.

## 4. Security Guardrails

- S3 Block Public Access is enabled on the upload bucket.
- S3 bucket encryption uses AWS-managed server-side encryption.
- DynamoDB encryption at rest is enabled by default.
- IAM role permissions are scoped to the specific DynamoDB table, S3 bucket, and CloudWatch log group.
- No secrets should be hardcoded in Lambda environment variables or Terraform files.
- CORS is restricted to the configured frontend origin.
- API authentication is represented as a placeholder. Do not store sensitive customer data behind an unauthenticated API.
- Public access should flow through API Gateway. Uploaded objects should not be directly public.

## 5. Terraform Starter Kit

The following Terraform is a compact starter kit. Split it into separate files as the project grows.

```hcl
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    archive = {
      source  = "hashicorp/archive"
      version = "~> 2.4"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

variable "aws_region" {
  type    = string
  default = "us-east-1"
}

variable "project_name" {
  type    = string
  default = "taskpulse"
}

variable "environment" {
  type    = string
  default = "dev"
}

variable "monthly_budget_usd" {
  type    = string
  default = "50"
}

variable "budget_alert_email" {
  type        = string
  description = "Email address that must confirm budget notifications."
}

variable "allowed_cors_origin" {
  type        = string
  description = "Frontend origin allowed to call the API, for example https://app.example.com."
}

locals {
  name = "${var.project_name}-${var.environment}"
}

data "archive_file" "lambda_zip" {
  type        = "zip"
  source_file = "${path.module}/lambda/index.js"
  output_path = "${path.module}/build/lambda.zip"
}

resource "aws_dynamodb_table" "app" {
  name         = "${local.name}-app"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "pk"
  range_key    = "sk"

  attribute {
    name = "pk"
    type = "S"
  }

  attribute {
    name = "sk"
    type = "S"
  }

  tags = {
    Project     = var.project_name
    Environment = var.environment
  }
}

resource "aws_s3_bucket" "uploads" {
  bucket = "${local.name}-uploads-${data.aws_caller_identity.current.account_id}"

  tags = {
    Project     = var.project_name
    Environment = var.environment
  }
}

data "aws_caller_identity" "current" {}

resource "aws_s3_bucket_public_access_block" "uploads" {
  bucket                  = aws_s3_bucket.uploads.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "uploads" {
  bucket = aws_s3_bucket.uploads.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_cloudwatch_log_group" "api" {
  name              = "/aws/lambda/${local.name}-api"
  retention_in_days = 14
}

resource "aws_iam_role" "lambda" {
  name = "${local.name}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy" "lambda" {
  name = "${local.name}-lambda-policy"
  role = aws_iam_role.lambda.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "${aws_cloudwatch_log_group.api.arn}:*"
      },
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:UpdateItem",
          "dynamodb:Query"
        ]
        Resource = aws_dynamodb_table.app.arn
      },
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]
        Resource = "${aws_s3_bucket.uploads.arn}/*"
      }
    ]
  })
}

resource "aws_lambda_function" "api" {
  function_name    = "${local.name}-api"
  role             = aws_iam_role.lambda.arn
  runtime          = "nodejs20.x"
  handler          = "index.handler"
  filename         = data.archive_file.lambda_zip.output_path
  source_code_hash = data.archive_file.lambda_zip.output_base64sha256
  timeout          = 10
  memory_size      = 256

  environment {
    variables = {
      TABLE_NAME     = aws_dynamodb_table.app.name
      UPLOAD_BUCKET  = aws_s3_bucket.uploads.bucket
      CORS_ORIGIN    = var.allowed_cors_origin
      ENVIRONMENT    = var.environment
    }
  }

  depends_on = [aws_cloudwatch_log_group.api]
}

resource "aws_apigatewayv2_api" "http" {
  name          = "${local.name}-http-api"
  protocol_type = "HTTP"

  cors_configuration {
    allow_origins = [var.allowed_cors_origin]
    allow_methods = ["GET", "POST", "PUT", "DELETE", "OPTIONS"]
    allow_headers = ["content-type", "authorization"]
    max_age       = 300
  }
}

resource "aws_apigatewayv2_integration" "lambda" {
  api_id                 = aws_apigatewayv2_api.http.id
  integration_type       = "AWS_PROXY"
  integration_uri        = aws_lambda_function.api.invoke_arn
  payload_format_version = "2.0"
}

resource "aws_apigatewayv2_route" "proxy" {
  api_id    = aws_apigatewayv2_api.http.id
  route_key = "ANY /{proxy+}"
  target    = "integrations/${aws_apigatewayv2_integration.lambda.id}"
}

resource "aws_apigatewayv2_stage" "default" {
  api_id      = aws_apigatewayv2_api.http.id
  name        = "$default"
  auto_deploy = true
}

resource "aws_lambda_permission" "api_gateway" {
  statement_id  = "AllowExecutionFromApiGateway"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.api.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.http.execution_arn}/*/*"
}

resource "aws_cloudwatch_event_rule" "daily_missing_checkins" {
  name                = "${local.name}-daily-missing-checkins"
  description         = "Runs once per day to detect missing weekly check-ins."
  schedule_expression = "rate(1 day)"
}

resource "aws_cloudwatch_event_target" "daily_missing_checkins" {
  rule      = aws_cloudwatch_event_rule.daily_missing_checkins.name
  target_id = "${local.name}-api-worker"
  arn       = aws_lambda_function.api.arn

  input = jsonencode({
    source = "eventbridge"
    task   = "detect_missing_checkins"
  })
}

resource "aws_lambda_permission" "eventbridge" {
  statement_id  = "AllowExecutionFromEventBridge"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.api.function_name
  principal     = "events.amazonaws.com"
  source_arn    = aws_cloudwatch_event_rule.daily_missing_checkins.arn
}

resource "aws_budgets_budget" "monthly" {
  name         = "${local.name}-monthly-budget"
  budget_type  = "COST"
  limit_amount = var.monthly_budget_usd
  limit_unit   = "USD"
  time_unit    = "MONTHLY"

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 50
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = [var.budget_alert_email]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = [var.budget_alert_email]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 100
    threshold_type             = "PERCENTAGE"
    notification_type          = "FORECASTED"
    subscriber_email_addresses = [var.budget_alert_email]
  }
}

output "api_endpoint" {
  value = aws_apigatewayv2_api.http.api_endpoint
}

output "dynamodb_table_name" {
  value = aws_dynamodb_table.app.name
}

output "uploads_bucket_name" {
  value = aws_s3_bucket.uploads.bucket
}
```

Minimal Lambda handler at `lambda/index.js`:

```javascript
exports.handler = async (event) => {
  return {
    statusCode: 200,
    headers: {
      "content-type": "application/json",
      "access-control-allow-origin": process.env.CORS_ORIGIN
    },
    body: JSON.stringify({
      ok: true,
      service: "TaskPulse",
      path: event.rawPath || "/"
    })
  };
};
```

## 6. Deployment Steps

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan \
  -var="budget_alert_email=founder@example.com" \
  -var="allowed_cors_origin=https://app.example.com"
```

Before applying:

- Confirm the plan does not include NAT Gateway, EKS, OpenSearch, RDS, EC2, or GPU resources.
- Confirm the budget alert email is correct.
- Confirm S3 public access block resources are present.
- Confirm CloudWatch log retention is set.

Apply only after review:

```bash
terraform apply \
  -var="budget_alert_email=founder@example.com" \
  -var="allowed_cors_origin=https://app.example.com"
```

Then smoke test:

```bash
API_ENDPOINT="$(terraform output -raw api_endpoint)"
curl "$API_ENDPOINT/health"
```

## 7. Validation Commands

```bash
terraform validate
terraform plan
```

Check S3 public access block:

```bash
aws s3api get-public-access-block \
  --bucket "$(terraform output -raw uploads_bucket_name)" \
  --region us-east-1
```

Check DynamoDB billing mode:

```bash
aws dynamodb describe-table \
  --table-name "$(terraform output -raw dynamodb_table_name)" \
  --region us-east-1 \
  --query "Table.BillingModeSummary.BillingMode"
```

Check CloudWatch log group retention:

```bash
aws logs describe-log-groups \
  --log-group-name-prefix "/aws/lambda/taskpulse-dev-api" \
  --region us-east-1 \
  --query "logGroups[].retentionInDays"
```

Check API endpoint:

```bash
curl "$(terraform output -raw api_endpoint)/health"
```

## 8. Troubleshooting

- Missing AWS credentials: run `aws sts get-caller-identity` and configure the correct profile.
- Region mismatch: ensure Terraform provider region and AWS CLI region are both `us-east-1`.
- Lambda ZIP build error: create `lambda/index.js` before running `terraform plan`.
- CORS error: set `allowed_cors_origin` to the exact frontend origin instead of `*`.
- Budget email not received: check the subscriber inbox and confirm the notification if AWS requests confirmation.
- Unexpected expensive resources: stop before `terraform apply` and remove any resource that introduces NAT Gateway, EKS, OpenSearch, RDS, EC2, or GPU.
- API Gateway integration issue: confirm `aws_lambda_permission.api_gateway` exists and references the HTTP API execution ARN.

## 9. Rollback and Destroy

To destroy resources:

```bash
terraform destroy \
  -var="budget_alert_email=founder@example.com" \
  -var="allowed_cors_origin=https://app.example.com"
```

If S3 deletion fails because the bucket contains files:

```bash
aws s3 rm "s3://$(terraform output -raw uploads_bucket_name)" --recursive
terraform destroy \
  -var="budget_alert_email=founder@example.com" \
  -var="allowed_cors_origin=https://app.example.com"
```

After destroy:

```bash
aws sts get-caller-identity
aws budgets describe-budgets --account-id "<account-id>"
aws ce get-cost-and-usage \
  --time-period Start=YYYY-MM-DD,End=YYYY-MM-DD \
  --granularity MONTHLY \
  --metrics UnblendedCost
```

## 10. Well-Architected Alignment

- Operational Excellence: Terraform, validation commands, logs, smoke tests, and rollback steps make the launch repeatable.
- Security: S3 public access is blocked, IAM permissions are scoped, secrets are not hardcoded, and CORS is restricted.
- Reliability: Serverless managed services reduce operational burden for a small team, and API/Lambda/DynamoDB can handle MVP traffic without server management.
- Performance Efficiency: Lambda, API Gateway HTTP API, and DynamoDB on-demand scale with usage instead of requiring upfront capacity planning.
- Cost Optimization: The design avoids always-on compute, NAT Gateway, EKS, OpenSearch, and RDS; it also adds budget alerts and short log retention.
- Sustainability: Serverless scaling and managed services reduce idle capacity for low-traffic MVP workloads.
