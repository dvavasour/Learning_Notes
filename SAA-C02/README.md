# AWS-SA-Notes
> Some notes from AWS SA Assoc revision. These are my notes, and probably won't be useful to anybody else, but are not private so I've left them `public`

_Let's bung all the notes in here and properly order them later_

# Access Management

For parts of access management

- The Principal (person or app requiring access)  
- Authentication (showing the principal matches an identity)
- Identity (The role you'll take to use access)
- Authorization (What this identity is allowed or forbidden to do)

# Definitions

- __High Availability__ - fast recovery
- __Fault Tolerance__ - operate through the failure

# WAF Pillars

- Operational Excellence
- Security
- Reliability
- Performance Efficiency
- Cost Optimisation


# IAM

Groups are not identities: so cannot be reference as a principal.
## IAM Limits

- 5000 users per account
- 10 groups per user
- 10 managed policies per user
- 2048 characters for all the inline policies for a user!
- 2 Access keys for a user (uncluding inactive ones)
- 300 groups per account (can be increased with ticket)


# Organizations

Master Account -> _Management Account_

- The master account cannot be restricted in any way
- Service Control Policies can be applied to organisation root, and OU or an account

# EC2
## Instance Families

- General Purpose
- Compute Optimised
- Memory optimised
- Storage Optimised
- Accelerated Computing

## Storage

- Non-ssd can't be used as boot volume
- Maximum for a single EBS volume is 64,000 IOPS
- Maximum for a EC2 instance is 80,000 IOPS of EBS (Instance store gives more)

## Instance Roles

Although the console does it all for you, when you create a role for an instance to assume, you also have to create an instance profile. It is actually the instance profile that is linked to the EC2 instance.
When you've applied the instance profile, it is available inside the instance via instance metadata.

## Partition Groups

- Cluster Placement Groups - keep instances together, ideally on the same host
- Partition Placement Group - keep instances apart within AZ, distributing between partitions
- Spread Placement Group - allows only one instance per partition

## Reservations

- Zonal reservation is for a specific instance size in a specific AZ. Capacity is reserved
- Regional reservation is for capacity, so applies to all sizes in all AZs. Capacity is not reserved



# Serverless
## Lambda

- A Lambda function can run for up to 15 minutes
- You need to specify the runtime (e.g. node.js or python)

## API Gateway

- API Gateway can give direct access to DynamoDB

- API Gateway Cross Origin Resource (CORS) Needed when API requests cross domains - __Read up on this__

## State Machines

- Configured in ASL (Amazon States Language)
- State machines can run for up to a year


# Containers

> Installing packages on Amazon Linux: amazon-linux-extras install docker (for example)

- Fargate "Task Role" is the way you giver permissions to containers.


# Networking

- L4 (Transport) tcp/udp etc. Technically if you reference ports, you're working at L4
- L5 (Session) - an open tcp connection. Allows statefulness
- L6 (Presentation) - e.g. TLS

### Reserved IPs

- __127.0.0.0/8__ - Loopback
- __169.254.0.0/16__ - Auto-configuration addressen

### Routeing

- Local
- Known
- Unknown (default route)

### VPCs and Subnets

- Maximum of /16, minimum of /28
- Default VPC: /16, with a /20 subnet in each AZ and IGW, open NACL, closed SG
- No public DNS entry for public IP address on a custom VPC unless enabled at the VPC level

### NAT GW and Instance

- NAT Instance is an EC2 Instance
- On a NAT Instance you *must* disable source/dest check in the Instance Actions
- NAT GW must be in PUBLIC subnet with a static elastic IP
- NAT GW is not HA - it's an instance in a subnet
- [https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html]

Using a NAT GW:

- Add a default route off the private subnet to the nat-gw
- Use a NAT GW in the same AZ
- NAT GW allows 5Gb/s autoscaling up to 45Gb/s
- NAT GW supports udp, tcp, icmp



`ssh-add -c <keyname>.pem`  (use `-K` on Mac)

`ssh -A -i <keyname>.pem <destination>`

Allows jump straight from bastion host to private host without transferring key to bastion.

For PuTTY, use the Auth option _Allow agent forwarding_

### NACLs

- ACL rules are applied in numerical order until a match is found
- There's a __*__ rule as a catch all
- While default VPC NACLs are open, custom NACLs are created with just implicit deny all
- NACLs allow explicit deny for a smaller address range (e.g. /32 for a dodgy host)
- NACLs are at L4, SGs are at L5

## VPC Peering

- a VPC connection is like an Internet Gateway
	+ connects two VPC - not a switch for more than two
	+ No IPV6 between regions
- VPC peering allows you to reference remote security groups if the peer's in the same region, even between accounts

### VPC Endpoints

- Gateway Endpoints for S3, DynamoDB, Interface Endpoints for others
	+ Gateway endpoints added to subnet route tables
	+ GW endpoints can have IAM policies applied
	+ GW endpoints behave like an IGW
- Interface Endpoints behave like an EC2 instance
	+ IF endpoints are allocated into a specific subnet
	+ IF endpoints can have security groups attached
	+ IF endpoints create:
		* DNS name for specific endpoint (i.e. by AZ)
		* DNS name for "nearest" endpoint <-- Use this one
		* If you set "Enable Private DNS Name" on the IF endpoint, it overrides the public DNS entry to point to the IF endpoint


# DNS
## DNS Private Zones
- Are associated with a specific VPC
- Must have enable DNS resolution AND enable DNS hostnames set on the VPC

Alias records can exist at the apex of your domain, CNAME records can't.

## Routeing Policies
- Simple routeing: `A` record with multiple entries randomly ordered according to `TTL`
- Failover routeing: Define primary and secondary A records: only one of each
- Weighted routeing: Each record has a set ID. Use weighted routeing for canary testing. NOT for load balancing
- Latency routeing: Doesn't measure latency, consults a lookup table, but not necessarily distance
- Geolocation: ONLY returns results if there's a geo match. Goes from most specific to least specific. If no default record, returns NOT FOUND
- Multivalue Answer: add individual entries with health checks, returns random set of healthy answers



# S3

- By default S3 bucket only trusts the owning AWS account
- IAM policies can grant permissions to roles, users, groups within the same account
- Explicit deny on IAM policy OR Bucket policy applied first.
- ACLs are legacy - use bucket policy to control access to objects or prefixes
- Block Public access on bucket overrides object ACLs
- https://aws.amazon.com/blogs/security/iam-policies-and-bucket-policies-and-acls-oh-my-controlling-access-to-s3-resources/

- Single part upload limited to 5GB
- Recommend Multipart over 100MB, essential over 5GB
- aws cli uses multipart by default
- multipart splits into up to 10,000 parts

## Encryption

- Client side (don't do this)
- Client managed keys SSE-C - you must present keys with every PUT and GET
- S3 managed encryption SSE-S3
	+ S3 manages keys for you
	+ Encryption: AES-256 is a telltale
	+ Same permissions for bucket/object and keys
- KMS SSE-KMS - allows separate permissions on object and key
	+ Encryption: AWS-KMS is telltale
	
Bucket defaults give a default level of encryption if nothing is specified for the object
Bucket policies can enforce the level/type of encryption allowed

## Website Hosting

- Need to both make bucket public and apply a bucket policy to allow `"*"` to have GET access on objects.
- CORS: Cross origin Resource Sharing

#### Create a pre-signed URL from the command line

`aws s3 presign s3://<bucketname>/<object-path>`

Need to check on options - expiry times etc.

- Pre-signed URLs are tied to the person who created the, so if their access is removed the pre-signed URL stops working
- Similarly, if you create a pre-signed URL using a Role, it will expire as soon as the role's credentials are rotated

## Storage Classes

- First byte latency - time from placing a request to when the object start coming to you. Milliseconds for Standard
- Lifecycle policies in a bucket can be filtered by prefix or tag
- Lifecycle policies can apply to current version, previous version or both
- You can define to expire previous versions of objects after a certain time
- Replication needs versioning on both ends
- Replication can be filtered by prefix or tag
- Replication is NOT retrospective, only applies to newly created objects
- SSE-C objects aren't replicated, SSE-S3 objects are, SSE-KMS not by default, can be enabled
- SSE-KMS objects are decrypted at source and re-encrypted at destination using a (region specific) KMS key at the destination as specified
- Intelligent tiering has a monthly "automation and monitoring" cost

## Cloudfront

- Distributions are Web and RTMP (streaming media)
- Cloudfront origin server MUST be publicly accessible
- OAI == Origin Access **Identity** | Identifier
	+ CF Distribution presents an OAI
	+ Bucket policy update to allow access only with this OAI
	+ Only application where the origin is S3 (not for EC2 or on-prem)

## EFS

- Tied to a VPC
- Only one mount target per AZ
- Performance modes are "General Purpose" and "Max I/O"
- Throughput modes are "Bursting" and "Provisioned" aka Throughput mode
- `yum install amazon-efs-utils` (will work without, but gives "tighter integration")
	+ You can use a filesystem type `efs`
	+ You can use the ""File System ID" as the hostname portion in the mount statement
- By default the mounted filesystem is only writeable by root

# Other Filesystems

- FSx for Windows (SMB, allows DFS)
- FSx for Lustre (high performance)


# Databases
## RDS

- snapshot backups are created manually, deleted manually, durable
- Automatic Backups
	- Maximum retention for automatic backups is 35 days
	- Automatic backups don't survive DB deletion
	- Log backups are every 5 minutes (allowing Point in Time restore)
- Restore is to a new DB instance NOT over the running DB and has a new CNAME
	+ If it's encrypted then you'll need to deal with regional key variations
	+ A restored DB will have its own SG
- Multi-AZ Mode
	+ Multi-AZ mode replication is synchronous
	+ Standby is inaccessible
	+ Some outage as CNAME is swapped over
	+ Modifications are applied to SB, then failover
	+ Only two instances: 1× primary, 1× secondary
- Read Replicas
	+ Read Replica can be Multi-AZ
	+ AWS handled network fnurgling for cross-site read replication
	+ There MUST be automatic backup available on the master DB
	+ RR can be promoted to master - much faster than restoring a snapshot
	+ You can have RR of an RR, but it introduces lag 
	+ RRs are NOT SYNCHRONOUS - there's a lag
	+ RRs are addressed individually, each has its own CNAME

## Aurora
### Aurora Provisioned

- Billed for storage used (high water mark). t or memory optimised instances, no "m"
- Backtrack must be enabled when the DB cluster is created
- Writer instance is allowed to write to storage
- A cluster can have up to 15 replicas using the same shared storage (same region, different AZs)
- Storage replication is synchronous over 3 or more AZs, so read replicas are synchronous
- Tiered for failover from tier-0 to tier-15
- Replicas come up quickly because no data copying
- Read CNAME points to all replicas
- And each node has its own endpoints
- And you can have custom endpoints
- Snapshots are of an instance
- Backups are of the cluster
- Backtrack takes the DB offline as "database storage" is rolled back
- Maximum window for Backtrack is 72 hours
- Parallel query must be enabled at DB creation
- Global database gives ~1sec latency to a separate region for read instance
- Global can't be run on "t" instances
- Maximum storage volume is 64TiB

### Aurora Serverless

- Set max and min capacity units
- Can set it to spin down after a period of inactivity
- Exists within a VPC (unlike S3 or Dynamo)
- Can restore a snapshot from Aurora Provisioned to Aurora Serverless
- Presented via a "VPC Endpoint" so can't be accessed from a VPN or inter-region VPC peer
- Query editor uses Data API - allows interactive queries on console SERVERLESS ONLY

## DynamoDB

- Tables, Rows, Columns -> Tables, Items Attributes
- Partition Key == Hash Key, Sort Key == Range Key
- DynamoDB is regionally based
- table name must be unique in your region in your account -> goes into the table's ARN
- Keys, attribute names, attribute values added together must be less than 400KB
- Reads and writes are for the __whole item__
- No resource policies - you must give a user or role permission to the table
- When the API returns http:200, your data is resiliently stored (at least 3 copies)
- Scan returns every item in the table. Filtering happens after a whole table scan
- Query can only query on keys, then filters the returned items. Only a single partition key can be queried
- Point in time recovery up to 35 days, enabled per table. Restore to a new table
- Encryption on by default, KMS allows role separation
- Global replication is multi-read/write - "last write wins". Must be empty when replication set up

### Consistency

- Strongly consistent reads are taken from the lead node
- Eventually consistent (the default, faster and half the cost of strongly consistent) reads are taken from any node
- Change between provisioned and on-demand mode can be made once per 24 hours

### Capacity Units

- Read Capacity Unit is 4KB or part
- Write Capacity Unit is 1KB or part
- Eventually Consistent reads (the default) use 0.5RCU

### Other

- Stream is always 24 hours capacity
- Stream Views
	+ KEYS_ONLY
	+ NEW_IMAGE (whole item after change)
	+ OLD_IMAGE (whole item before change)
	+ NEW\_AND\_OLD_IMAGES (whole item both before and after change)
- Item being added to stream can be a Lambda trigger

### Indexes

- Global and Local Secondary Indexes (GSI and LSI)
- LSI
	- LSI allows you to reference an alternative Sort Key
	- LSI must be created at the same time as the table
	- You can still only query using an exact match for the the Partition Key
	- Up to 5 LSIs per table (check this is still current)
	- Shares capacity units with the table
- GSI
	+ You specify an alternate Partition Key from the data
	+ Up to 20 GSIs per table (can be increased, check for currency)
	+ GSI doesn't need a Sort Key
	+ GSI has its own capacity units
	+ GSI is essentially a replica table - so reads can only be eventually consistent with written table
	+ GSI can be created against an existing table (as it's just a replica)
	+ Indexes have "Projected Attributes" - which of the attributes in the source items get copied to the index
	
### Transactions

DynamoDB can assemble multiple operations as a transaction - it takes more resource as it's Prepare->Commit. Up to 25 items or 4MB.

# Load Balancing

- An LB node is placed in each AZ, and DNS splits traffic equally between them

## Classic LB

- Deprecated - only recommended for EC2 Classic
- Single SSL certificate per LB
- Works at L3/L4
- Internal LB doesn't have public IP address
- Health checks are: http, https, tcp, ssl


## Application LB

- uses target groups (as does a NLB)
- must be deployed in at least two subnets

## Network LB

- is IP addressable
- uses target groups

# Auto-scaling
## Launch Configuration

- Cannot be modified
- Can't launch an instance outside an ASG

## Launch Template

- Cannot be modified
- Has versioning
- Has inheritance
- Much richer options set
- Can launch an instance outside an ASG

## ASG

- Scaling policies are:
	+ **Simple Scaling:** based on an action when an event happens, e.g. CPU level breach. _has a cooldown period_
	+ **Target Scaling:** based on actions to bring a metric to its target
	+ **Stepped Scaling:** for adding/removing in blocks
- Use the utility `stress` to max out CPUs for ASG testing

```
sudo amazon-linux-extras install epel -y
sudo yum install stress -y
stress --cpu 2 --timeout 30000
```


# ID Federation

- ADFS is Active Directory Federation Service

# Integrations

## SNS

- Regional service, resilient
- Endpoints are public - so VPC endpoint is needed
- SNS Topics have their own resource policies with specified principal
- Messages are <= 256KB
- SNS now has SMS output and mobile platforms
- Remember SQS Fanout

## SQS

- Maximum message length is <= 256KB
- Short Polling returns immediately if empty
- Long Polling blocks for *WaitTimeSeconds*
- Messages are hidden once retrieved for _VisibilityTimeout_ seconds. Processing node is responsible for deleting messages
- Standard Queues have unlimited scaling, deliver out-of-order and may duplicate
- FIFO Queues deliver in order no more than once



## DAX (DynameDB Accelerator)

- In memory cacheing, eventually consistent NOT strongly consistent
- Has an *item cache* and a *query cache*

## Elasticache

- `redis` or `memcached`
- often used for storing session state

## Snowball

### Snowball

- Storage only
- Transfer data in using EITHER
	+ Snowball client (on your client machine)
	+ S3 gateways - installed on the SB makes it an S3 device
	+ As standard 80TB of raw storage, 72TB useable

### Snowball Edge

- Storage and compute
- Snowball edge can present NFS as well as S3

### Snowmobile

- uneconomic below 10PB, can go up to 100PB

## Storage Gateway

- It's a virtual appliance - e.g. a VMware appliance
- Three flavours
	+ File Gateway - presents SMB shares
	+ Volume Gateway - presents iSCSI volumes
		* Can snapshot a volume and present it as EBS
		* You could present volumes inside EC2 as iSCSI for DR
		* Gateway stored volumes are on the GW and snapshotted to S3
		* Gateway cached volumes - primary copy in S3, cached bits in SGW
	+ Virtual Tape Library
		* Move tapes to a "shelf" which is a S3 storage class


## Database migration service

- Adds a replication instance twixt source and target
	+ This allows schema conversion
	+ Also useful for peeling off part of a schema


## VPNs

- Customer Gateway is the device on your premises
- Virtual Private Gateway is attached to a VPC
- Can use either static routeing or BGP	
	+ Resilient connection (2×CGW, 2×tunnel endpoint) REQUIRES BGP
	+ Set up routes both in home router AND aws route tables
- Routeing hierarchy of preference:
	+ local routes (even over a more specific static route)
	+ static routes take preference over dynamic (propagated) routes
	+ dynamic routes learned from Direct Connect
	+ static routes from VPN
	+ dynamic routes learned from VPN

## Direct Connect

- Put your equipment in a DX location, and connect with a Letter Of Authorisation
- DX location terminates at public services and at a VGW in a VPC
- Run multiple VIFs (Virtual Interfaces) on top of the DX connection
	+ Public VIF connects to public resources (S3, Dynamo etc)
- DX is either 1Gb/s or 10Gb/s
- Latency for DX is consistently low
- DX is NOT encrypted
- DX is NOT HA
- Often a VPN provisioned while waiting for DX, then it becomes failover


# Monitoring
## Cloudwatch

Alarms have the following components

- **Metric:** Data points being measured
- **Threshold:** (may be static or anomaly detection)
- **Period:** How long in threshold state before generating alarm
- **Action:** SNS, Autoscale, EC2

### Cloudwatch Logs

Catch events. Namespace hierarchy is:

Log Group (e.g. service) --> Log Streams (identify instance or time bucket) --> Log Events

In Splunk terms:

> Log Group is roughly equivalent to sourcetype
> 
> Log Stream is closely equivalent to source
> 
> Event is very equivalent to event

Exports and streaming out is configured at the Log Group level, as well as retention policies.

### Cloudwatch Events

Accept events and take actions.
Can be schedule driven in crontab form
Action can be to stop EC2 instance

## Cloudtrail

Stored in "Event History" by default for 90 days


# KMS

- KMS can encrypt/decrypt up to 4KB items

# WAF

- Can front for:
	+ CloudFront
	+ Application Load Balancer
	+ API Gateway
- Actions can be:
	+ Allow all except specified
	+ Block all except specified
	+ Count requests that match a specification