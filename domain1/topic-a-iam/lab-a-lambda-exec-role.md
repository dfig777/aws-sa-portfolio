# Lab A — IAM: Lambda Execution Role (Domain 1)

## Objective
Create a Lambda execution role that follows least privilege and avoids long-lived credentials.

## What I built
- **IAM role:** `Flagship-LambdaExecRole-v1`
- **Trust principal:** `lambda.amazonaws.com`
- **Baseline permissions:** `AWSLambdaBasicExecutionRole` (CloudWatch Logs write)

## Why this matters (real-world)
AWS workloads should not embed access keys. A Lambda function should assume an IAM role at runtime and receive **temporary credentials** from STS. This reduces credential leakage risk and keeps permissions scoped to the workload.

## Key concepts demonstrated

### 1) Trust policy = who can assume the role
A trust policy is attached to the **role** and controls which principals can call `sts:AssumeRole`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Principal": { "Service": "lambda.amazonaws.com" }
    }
  ]
}
