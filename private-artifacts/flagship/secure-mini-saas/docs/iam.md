# IAM - Flagship

## Lambda execution role
Role: Flagship-LambdaExecRole-v1

### Trust relationship
- Principal: lambda.amazonaws.com
- Action: sts:AssumeRole

### Permissions
- AWSLambdaBasicExecutionRole (CloudWatch Logs)
- Flagship-DDB-UsersTable-CRUD (CRUD on DynamoDB table Flagship-Users only)
  - Resource: arn:aws:dynamodb:us-east-2:419142569739:table/Flagship-Users
