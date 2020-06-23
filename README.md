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

# IAM Limits

- 5000 users per account
- 10 groups per user
- 10 managed policies per user
- 2048 characters for all the inline policies for a user!
- 2 Access keys for a user (uncluding inactive ones0

)

# Organizations

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
- ACLs are legacy
- Block Public access on bucket overrides object ACLs
- https://aws.amazon.com/blogs/security/iam-policies-and-bucket-policies-and-acls-oh-my-controlling-access-to-s3-resources/

- Single part upload limited to 5GB
- Recommend Multipart over 100MB, essential over 5GB
- aws cli uses multipart by default

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
