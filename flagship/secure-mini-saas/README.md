# Secure Mini-SaaS (Flagship) — AWS SAA Portfolio

## Current milestone (Domain 1 / Topic A: IAM Foundations)
Built the IAM foundation for a serverless backend using role-based access and least privilege.

### What I implemented
- DynamoDB table: `Flagship-Users` (us-east-2)
- Lambda execution role: `Flagship-LambdaExecRole-v1`
  - Trust: Lambda service (`lambda.amazonaws.com`) can assume the role via STS
  - Permissions:
    - `AWSLambdaBasicExecutionRole` (CloudWatch Logs)
    - `Flagship-DDB-UsersTable-CRUD` (scoped CRUD to only `Flagship-Users` table ARN)

### Why this matters (security)
- No long-lived AWS keys in code (workloads use STS temporary credentials)
- Clear separation: trust policy (who can assume) vs permissions (what can be done)
- Least-privilege access to a single DynamoDB resource

