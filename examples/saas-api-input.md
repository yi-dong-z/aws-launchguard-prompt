# Example Input: SaaS API MVP

Use the AWS LaunchGuard prompt with this user input.

---

I am building a SaaS API MVP called TaskPulse.

TaskPulse helps small product teams collect weekly status updates from team members and generate a lightweight project health summary. The first version needs:

- A public HTTPS API.
- A database for teams, users, weekly check-ins, and generated summaries.
- File uploads for optional CSV imports.
- A placeholder for authentication, but I may add Cognito later.
- A daily background job to detect missing check-ins.
- Basic logs for debugging.

Constraints:

- AWS region: us-east-1
- Monthly budget ceiling: USD 50
- Environment: dev/staging
- Expected traffic: fewer than 1,000 users and fewer than 100,000 API requests in the first month
- Data sensitivity: non-regulated customer data, but it should not be public
- IaC preference: Terraform

Please generate the AWS LaunchGuard output.
