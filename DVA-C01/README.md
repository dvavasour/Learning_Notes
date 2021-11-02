# IAM
## AWS Inspector
Freestanding service that runs across a VPC and creates a report on what's open and accessible

*IAM Policy Simulator* show which policies allowed or denied actions.

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

Can have a secure listener on ELB using secure port on EC2 where the SSL is terminated.

## Route 53
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
- No data typing or sorting
- *lazy loading* = check the cache, then access origin if cache miss

Elasticache - Redis
- Allows lists, hashes and data typing
- Allows sorting
- Allows Multi-AZ and persisted

## Parameter Store
- Can be hierarchical
- Secrets Manager fosts money and provides secret rotation, Paramater store is free and doesn't


# S3
Max object size is 5TB  

URL Format:
`https://<bucket-name>.s3.<region>.amazonaws.com/<object-name>`

There's now Glacier and "Glacier Deep Archive"
Cloudfront has signed URLs
To force https on Cloudfront, use `Viewer Protocol Policy`  
Use Multipart Upload for files bigger than 100MB


# Serverless
## Lambda
Services that can invoke Lambda:
- DynamoDB, Kinesis, SQS, SNS, SES, API Gateway and others.

1,000 concurrent executions per second  
Maximum execution duration for request is 900 seconds
http 429 is limit exceded  
Capacity is scaled from memory size
Allow Lambda to access resources in a VPC:
- Uses an elastic IP (function execution role must haver ENI permissions)
- Security group is allocated to the function
- Put connect strings in the function

There's a package size limit. Get round this using layers, a `.zip` file containing extra stuff. 

## API Gateway
10,000 requests per second, 5,000 concurrent requests *per region*  
Default cacheing is 300 seconds
Import APIs using OpenAPI (Swagger)  
SOAP is passthrough, API can convert XML response to JSON  
To defeat cacheing, need to:
- `Cache-Control: max-age=0` http header
- Sign to show permission

General Rule: 4XX errors are client side, 5XX errors are server side
WebSocket API allows 2-way communication, not just the REST request->reply
*Stage Variables* can be used to allow a single API gateway to be used for different contexts (e.g. prod, dev)

## Step Functions
Copious logging

## X-Ray
X-Ray daemon must be installed on EC2 instance and SDK must be used  
In ECS, X-ray daemon should be in a different container  
X-Ray annotations are KV pairs to spoon feed values.

## Other
AWS AppSync is a thing. https://aws.amazon.com/appsync/

# DynamoDB
Document and KV: JSON, HTML, XML  
Does offer a strongly consistent version with ACID as well as eventually consistent `TransactWriteItems`  
Tables, items (rows), attributes (columns)  
Primary Key: partition key; composite key (partition key + sort key, e.g. User ID + message id)  
Maximum item size is 400KB

IAM: there's a condition `dynamodb:LeadingKeys` - allows access to items where partition key matches the user's `User_ID`  

Local Secondary Index
- Same partition key, different sort key
- Most be created when table created

Global Seconday Index
- Different partition key and sort key to table
- If you're going to use a low cardinality sort key, create a random high cardinality partition key
- Can be created at any time
- initial quota of 20 per table, support ticket to increase

Query
- Find items in table using primary key
- Results sorted by primary key
- set `Scan IndexForward` to FALSE for reverse ordering

Scan
- scans every table item
- returns all attributes by default
- `ProjectionExpression` refines results
  - Make scan more efficient by seting smaller page size
  - Isolate scans to specific tables
  - Parallel scans (for table bigger than 20GB)
  - *Don't use scan!* - prefer `Query`, `Get` or `BatchGetItem` APIs

## Throughput
Capacity Units
- Write Capacity Unit = 1× 1KB/s
- Strongly Consistent Read = 1× 4KB/s
- Eventually Consistent Read (default) = 2× 4KB reads per second

On-Demand Capacity available, but provisioned enables cost control
Provisioned throughput limits can be raised by a support ticket
Use exponential backoff client side, especially if client is AWS service.

## DAX
Improves resonse for eventually consistent reads only  
Point API calls at DAX explicitly  

## TTL
Good for session data, etc

## DynamoDB Streams
24 hours only  
Good Lambda Event Source  

# KMS
## CMK Customer Master Key
- Can add an alias for easy reference
- Has a state (enabled, pending deletion, etc)
- Key material either your own or AWS provided
- Cannot export
- Limited to 4KB of payload (typically a session key/data key)
- AWS Managed is the keys automatically created for e.g. S3. Cannot be used by the customer
- Keys are region specific

## KMS
- `aws kms encrypt|decrypt|re-encrypt|enable-key-rotation|generate-data-key`
- KMS is multi-tenant, CloudHSM uses dedicated hardware

# Other Services
## SQS
- Standard and FIFO
- Standard: Default; unordered; may duplicate; "delivered at least once"
- FIFO: Strict ordering; delivered once
- Visibility Timeout: default 30s; max 12 hours `ChangeMessageVisibility`
- Retention Period: default 4d; min 1min, max 14d `SetQueueAttributes`
- Short Polling returns and bills; Long Polling returns on message or timeout
- Delay Queue: invisible for up to 900s
- Large SQS is >256KB; <2GB - stores in S3, need *SDK for Java* and *SQS Extended Client Library for Java*

## SNS

## Kinesis
- Streams (real time); Firehose (near real time); Analytics
- Streams made of Shards (capacity unit); client should have a (or less than one) processor per shard
- Events are given a sequence number on being written. This can be used for consumption.

## Elastic Beanstalk
- Java, PHP, Python, Ruby, Go, Docker, .NET, Node.js
- Apache, Tomcat, Passenfer, Puma, Nginx, IIS
- Provisions EC2, RDS, S3, ELB, ASG et al
- YAML or JSON config file, `<filename>.config`, `.ebextensions/` folder at top level of bundle
- Deployment types: All at Once (interruption); Rolling; Rolling with Additional Batch; Immutable (full deployment and redirect); Traffic Splitting (for canary testing)
- RDS: Inside EBS for dev/test (destroyed at EB teardown); Outside EBS needs security config and connect strings


# Monitoring
- Cloudwatch agent for OS stuff
- Cloudwatch; CW Alarms; CW Logs
- Cloudtrail records AWS API calls
- Cloudwatch=Performance, Cloudtrail=Audit
- Cloudwatch Detailed Monitoring is every minute. Less needs a custom metric.

Cloudwatch alarm statuses: `OK`, `ALARM`, `INSUFFICIENT_DATA` (i.e. unknown)
CloudWatch Agent will scrape logfiles inside instances

# Developer
- Continuous Integration - CodeCommit
- Continuous Delivery - CodeBuild, CodeDeploy
- Continuous Deployment - CodePipeline

## CodeDeploy
- Deployments either: In-Place (Rolling Restart); Blue/Green


`appspec.yml` in YAML (unless deploying to Lambda, json is then permissible)
Folder setup
- `apppspec.yml` - always in root directory of revision
- `/Scripts`
- `/Config`
- `/Source`

Lifecycle event hooks run in the *Run Order*
- Phase 1 Deregister from LB
  -  `BeforeBlockTraffic`
  -  `BlockTraffic`
  -  `AfterBlockTraffic`
- Phase 2 Deployment
  - `ApplicationStop`
  - `DownloadBundle`
  - `BeforeInstall`
  - `Install`
  - `AfterInstall`
  - `ApplicationStart`
  - `ValidateService`
- Phase 3 Reregister with LB
  - `BeforeAllowTraffic`
  - `AllowTraffic`
  - `AfterAllowTraffic`

## Codepipeline
- CI/CD Service
- Can be configured to trigger automatically on source code change
- Integrates with AWS & 3rd party tools (github, Jenkins)

## ECS
- Docker or Windows Containers
- Deploy to either **EC2 Cluster** or **Fargate** for serverless
- EC2 gives more control
- Images go in Elastic Container Registry
- Execution role is attached to the ECS Task

## Docker and Elastic Beanstalk
- Use EB to deploy container to single EC2 instance
- or multiple Docker instances to a EC2 cluster
- Deploy application by uploading code bundle to EB (and update with new code version)

## Cloudformation
- YAML or JSON
- Sections: Version, Description, Metadata, Parameters, Conditions (incl Mappings), Transform (e.g. Include), Resources, Outputs
- Only required section is Resources
- Set `DeletionPolicy` to `Retain` to keep resources after stack deletion
- `Fn::GetAtt` returns a resource's attribute

## SAM is a CF extension for Serverless with a simplified syntax
- `sam package` - mushes the CF YAML
- `sam deploy`
- sam is a separate CLI
- YAML file includes a `Transform:` clause telling CF it's a server less deployment

CF Nested Stacks allow code re-use

# Advanced IAM
## Web Identity Federation
- Cognito is identity broker twixt IAM and Google, FB, etc giving temporary credentials
- User authenticates with external service first
- Temp credentials map to IAM role
- User Pools, Sign-in, Identity Pools
- User Pool manages interface to (e.g.) FB, which provides a JWT token
- Identity Pool swaps this for AWS Credentials which map to IAM role
- **Push Synchronisation** synchronises between devices (SNS under the bonnet)

## Policies
- AWS Managed Policies (can't be changed)
- Customer Managed Policies (use just in your account, can copy AWS managed policy)
- Inline Policy: 1:1 Relationship with entity it's attached to.

## AssumeRoleWithWebIdentity
- Security Token Service API Call
- For use where you can't use Cognito
- Temporary Credentials - default timeout = 1 hour

## Cross-Account Access
- 
