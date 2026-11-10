# Terraform AWS Security Training Lab Index

This directory provides the curriculum sequence for the Terraform AWS Security Training series.

Each infrastructure lab is maintained in its own Git repository.

Repository names begin with the lab number, such as `01-`, so they sort in curriculum order on GitHub.

The labs are designed to build on one another while still explaining their individual architecture, security controls, costs, lifecycle, and cleanup requirements.

## Lab 0 - Security and Cost Foundations

Repository:

```text
00-terraform-aws-security-training
```

Link: [github.com/lshillingford/00-terraform-aws-security-training](https://github.com/lshillingford/00-terraform-aws-security-training)

Focus:

- Course standards
- HCP Terraform
- AWS authentication
- IAM and least privilege
- Terraform state
- Source-control safety
- AWS Budgets
- Cost management
- Tagging
- Naming
- Plan and destroy review

Lab 0 does not deploy the main training infrastructure.

It establishes the standards used by the rest of the series.

## Lab 1 - Network Foundation

Repository:

```text
01-terraform-aws-network-foundation-lab
```

Link: [github.com/lshillingford/01-terraform-aws-network-foundation-lab](https://github.com/lshillingford/01-terraform-aws-network-foundation-lab)

Status: released, version 1.0.2.

Focus:

- Amazon VPC
- CIDR planning
- Availability Zones
- Public subnets
- Private subnets
- Internet Gateway
- Route tables
- Public and private routing
- Terraform outputs
- Network ownership

Initial architecture:

```text
VPC
|-- Public subnet A
|-- Public subnet B
|-- Private subnet A
|-- Private subnet B
|-- Internet Gateway
|-- Public route table -> Internet Gateway
`-- Private route tables -> no default Internet route
```

NAT Gateway is disabled by default.

The lab outputs network identifiers for use by later labs.

## Lab 2 - EC2 and NGINX

Repository:

```text
02-terraform-aws-ec2-web-lab
```

Link: [github.com/lshillingford/02-terraform-aws-ec2-web-lab](https://github.com/lshillingford/02-terraform-aws-ec2-web-lab)

Status: released, version 1.0.0.

Focus:

- Amazon EC2
- Amazon Linux 2023
- Security groups
- Encrypted gp3 storage
- IMDSv2
- User data
- NGINX
- Terraform templates
- Consuming network inputs
- Application validation

The lab uses the Network Foundation as a prerequisite instead of silently creating another VPC.

SSH is disabled by default.

## Lab 3 - Secure S3 Object Storage

Focus:

- Amazon S3
- Private buckets
- S3 Block Public Access
- Encryption
- Versioning
- Lifecycle rules
- Globally unique naming
- Object-storage cost management

The standard architecture does not require a public S3 bucket.

## Lab 4 - Systems Manager Administration

Focus:

- AWS Systems Manager
- Session Manager
- IAM roles
- EC2 instance profiles
- Administrative access without SSH
- Reduced inbound exposure

This lab demonstrates that EC2 administration does not inherently require an Internet-exposed SSH service.

## Lab 5 - Application Load Balancer and Optional HTTPS

Focus:

- Application Load Balancer
- Target groups
- Health checks
- Security-group relationships
- Multiple Availability Zones
- AWS-generated load balancer hostnames
- Optional Route 53
- Optional AWS Certificate Manager

A purchased custom domain is not required for the basic learning path.

## Lab 6 - Private RDS and Application Data

Focus:

- Amazon RDS
- Private database subnets
- Database security groups
- Encryption
- Backups
- Application-to-database communication
- Sample application data
- Database cleanup and cost

The standard database architecture is not publicly accessible.

The application tier should connect to the database using controlled security-group relationships.

## Lab 7 - CloudFront and Private S3

Focus:

- Amazon CloudFront
- Private Amazon S3 origin
- Origin Access Control
- HTTPS
- Static application content
- Optional custom domains
- CDN cost considerations

The basic learner path can use the AWS-generated CloudFront HTTPS hostname.

## Lab 8 - HCP Terraform to AWS OIDC

Focus:

- HCP Terraform workload identity
- OpenID Connect
- AWS IAM trust policies
- AWS STS
- Temporary credentials
- Removal of long-lived training access keys

This lab represents the advanced authentication target for the curriculum.

## Lab 9 - Secure Three-Tier Capstone

Focus:

- Networking
- Load balancing
- Compute
- Object storage
- Relational databases
- IAM
- Encryption
- Administrative access
- HCP Terraform
- Terraform state
- Resource ownership
- Security validation
- Cost management
- Full infrastructure lifecycle

The capstone combines the earlier concepts into an integrated application architecture.

The goal is to validate real application connectivity rather than simply confirm that AWS resources exist.

## Optional Serverless Capstone

An optional advanced exercise may introduce:

- Amazon CloudFront
- Amazon S3
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB

This provides a serverless alternative to the traditional compute and relational-database architecture.

## Recommended Order

Complete the labs in sequence:

```text
Lab 0
  |
  v
Lab 1
  |
  v
Lab 2
  |
  v
Lab 3
  |
  v
Lab 4
  |
  v
Lab 5
  |
  v
Lab 6
  |
  v
Lab 7
  |
  v
Lab 8
  |
  v
Lab 9
```

Some labs build directly on infrastructure or concepts introduced earlier.

## Common Lifecycle

Infrastructure labs follow the same general workflow:

```text
Read
  |
  v
Configure
  |
  v
Plan
  |
  v
Review
  |
  v
Apply
  |
  v
Validate
  |
  v
Destroy Plan
  |
  v
Review
  |
  v
Destroy
  |
  v
Confirm Cleanup
```

Every billable lab includes cleanup as part of the learning exercise.

## Curriculum Goal

By the end of the series, learners should be able to explain not only how Terraform created AWS infrastructure, but also:

- Why each major resource exists
- Which resources are publicly reachable
- Which resources remain private
- How IAM permissions affect Terraform
- How Terraform state tracks resource ownership
- How tags identify lab resources
- How AWS costs accumulate
- How HCP Terraform manages remote execution and state
- How short-lived workload identity improves authentication
- How to safely review, validate, and destroy infrastructure
