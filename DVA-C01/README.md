# IAM
## AWS Inspector
Freestanding service that runs across a VPC and creates a report on what's open and accessible


# Serverless
## Lambda
Services that can invoke Lambda:
- DynamoDB, Kinesis, SQS, SNS, SES, API Gateway and others.

1,000 concurrent executions per second  
http 429 is limit exceded  
Allow Lambda to access resources in a VPC:
- Uses an elastic IP
- Security group is allocated to the function

## API Gateway
10,000 requests per second, 5,000 concurrent requests

## Step Functions
Copious logging

## X-Ray
X-Ray daemon must be installed on EC2 instance and SDK must be used  
