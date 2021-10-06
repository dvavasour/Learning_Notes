# IAM
## AWS Inspector
Freestanding service that runs across a VPC and creates a report on what's open and accessible

# EC2
## EBS
There's gp2 (up to 16,000 IOPS per volume), io1 (50 IOPS/GB up to 64,000 per volume) and the new io2 (500 IOPS/GB up to 64,000 per volume and 99.999% durability) for SSD  
There's st1 (Throughout optimized) and sc1 (Cold HDD) for HDD  
Snapshots are encrypted or unencrypted depending on volume status

## ELB
- Application Load Balancer (L7)  
- Network Load Balancer (L4) more expensive  
- Classic Load Balancer  
- Gateway Load Balancers (for 3rd party appliances)  
- X-Forwarded-For give the true source IP
- 504 error means origin server not responding

## R53
*Alias* records typically used to map zone apex  

## RDS
- SQL Server, Oracle, MySQL, PostgrSQL, Maria DB, Amazon Aurora
- For OLTP
- Use RedShift for OLAP

Automated Backup
- Includes journals
- Retention period up to 35 days
- Allows Point in Time recovery

Snapshot
- Stored indefinitely
- Kinda like IMP/EXP

Encryption
- Must be enabled at creation
- Includes backups, snapshots, replicas
- Integrates with KMS
- You can snapshot a database, encrypt then run as encrypted instance

Multi-AZ
- Exact copy for DR in another AZ

Read Replica
- Can be in same AZ

Elasticache - Memcached
- Simpler
- No persistence or multi-AZ
- No dataatyping or sorting

Elasticache - Redis
- Allows lists, hashes and data typing
- Allows sorting
- Allows Multi-AZ and persisted

## Parameter Store


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
10,000 requests per second, 5,000 concurrent requests *per region*  
Default cacheing is 300 seconds
Import APIs using OpenAPI (Swagger)  
SOAP is passthrough, API can convert XML response to JSON  

## Step Functions
Copious logging

## X-Ray
X-Ray daemon must be installed on EC2 instance and SDK must be used  
In ECS, X-ray daemon should be in a different container  
X-Ray annotations are KV pairs to spoon feed values.
