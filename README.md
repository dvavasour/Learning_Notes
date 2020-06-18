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

