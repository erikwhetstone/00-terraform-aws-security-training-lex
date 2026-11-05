# Terraform AWS Security Training

A security-first, cost-conscious, hands-on learning series for Terraform, HCP Terraform, and Amazon Web Services (AWS).

## About This Project

This training series is designed for IT administrators and engineers who want to learn infrastructure as code while also learning the operational practices required to manage cloud infrastructure safely.

The goal is not simply to provide Terraform code that deploys successfully.

Each lab explains:

- What is being built
- Why each AWS resource exists
- How Terraform manages the resource
- What security decisions are being made
- What the infrastructure is expected to cost
- How Terraform state is protected
- How to validate the deployment
- How to safely destroy the infrastructure

Security, cost awareness, resource ownership, and cleanup are part of every lab.

## Learning Environment

The training series uses the following workflow:

```text
GitHub Repository
        |
        v
HCP Terraform Workspace
        |
        v
AWS Training or Sandbox Account
```

GitHub stores Terraform source code.

HCP Terraform performs remote Terraform operations and stores shared Terraform state.

AWS provides the infrastructure used by the labs.

Environment-specific values and credentials must not be stored in public source repositories.

## Training Objectives

By completing the series, learners should gain practical experience with:

- Terraform configuration and workflow
- HCP Terraform workspaces
- Remote Terraform state
- AWS Identity and Access Management (IAM)
- Least-privilege access
- AWS networking
- Amazon EC2
- Amazon S3
- AWS Systems Manager
- Application Load Balancers
- Amazon RDS
- Amazon CloudFront
- Encryption
- Resource tagging and ownership
- AWS cost management
- Terraform plan, apply, validation, and destroy workflows
- Short-lived AWS authentication using workload identity
- Multi-tier AWS architecture

## Security Principles

The labs follow these baseline security principles:

- Do not use AdministratorAccess as the recommended training policy.
- Do not use SSH by default.
- Require IMDSv2 for Amazon EC2.
- Encrypt EBS and database storage.
- Keep Amazon S3 Block Public Access enabled.
- Keep Amazon RDS databases private.
- Keep HCP Terraform Auto Apply disabled.
- Verify the target AWS account before deployment.
- Review Terraform plans before applying them.
- Review destroy plans before confirming destruction.
- Store credentials outside Terraform source code.
- Introduce short-lived credentials and workload identity in the advanced authentication lab.

See [SECURITY.md](SECURITY.md) for the complete security standard.

## Cost Philosophy

Cloud training resources can generate real AWS charges.

The initial curriculum targets approximately **$10 or less in AWS list-price usage per learner** when labs are run for short training sessions and billable resources are destroyed promptly.

Cost controls include:

- A suggested AWS training budget of **$10**
- Suggested budget warnings at **$5** and **$8**
- NAT Gateway disabled by default
- Short-duration use of expensive resources
- Cost estimates for every billable lab
- A destroy exercise for every billable lab

Budget notifications are warnings. They do not automatically shut down AWS resources.

Free Tier benefits and AWS credits should not be assumed when estimating lab costs.

See [COSTS.md](COSTS.md) for the complete cost standard.

## Curriculum

### Lab 0 - Security and Cost Foundations

Establish the standards used throughout the training series before deploying infrastructure.

Topics include:

- Terraform and HCP Terraform workflow
- AWS authentication
- IAM and least privilege
- Terraform state security
- Git and source-control safety
- Standard variables
- Resource tagging
- AWS Budgets
- Cost awareness
- Plan and destroy review
- Public repository safety

### Lab 1 - Network Foundation

Build the reusable AWS networking foundation for later labs.

Topics include:

- Amazon VPC
- CIDR planning
- Availability Zones
- Public subnets
- Private subnets
- Internet Gateway
- Route tables
- Terraform outputs
- Network cost considerations

NAT Gateway is disabled by default to avoid unnecessary cost.

### Lab 2 - EC2 and NGINX

Deploy an Amazon Linux 2023 web server into the network created in Lab 1.

Topics include:

- Amazon EC2
- Security groups
- Encrypted EBS
- IMDSv2
- EC2 user data
- NGINX
- Terraform templates
- Network inputs
- Deployment validation

SSH is not enabled by default.

### Lab 3 - Secure S3 Object Storage

Build secure object storage with Amazon S3.

Topics include:

- Private S3 buckets
- S3 Block Public Access
- Encryption
- Versioning
- Lifecycle rules
- Terraform naming strategies
- Object-storage cost management

### Lab 4 - Systems Manager Administration

Learn to administer EC2 without opening SSH.

Topics include:

- AWS Systems Manager
- Session Manager
- IAM roles
- Instance profiles
- Secure administrative access
- Reducing inbound network exposure

### Lab 5 - Application Load Balancer and Optional HTTPS

Introduce highly available application delivery.

Topics include:

- Application Load Balancer
- Target groups
- Health checks
- Security-group relationships
- Multiple Availability Zones
- AWS-generated load balancer hostnames
- Optional Route 53 and ACM integration

A custom domain is optional.

### Lab 6 - Private RDS and Application Data

Add a relational database to the application architecture.

Topics include:

- Amazon RDS
- Private database subnets
- Database security groups
- Encryption
- Backups
- Application-to-database connectivity
- Sample application data
- Short-duration database labs and cleanup

The database is not publicly accessible.

### Lab 7 - CloudFront and Private S3

Build secure content delivery using Amazon CloudFront.

Topics include:

- Amazon CloudFront
- Private S3 origins
- Origin Access Control
- HTTPS
- Static application content
- Optional custom domains
- CDN cost considerations

A purchased domain is not required.

### Lab 8 - HCP Terraform to AWS OIDC

Replace long-lived AWS access keys with short-lived credentials.

Topics include:

- Workload identity
- OpenID Connect (OIDC)
- AWS IAM trust policies
- AWS STS
- Temporary credentials
- HCP Terraform authentication
- Credential lifecycle security

### Lab 9 - Secure Three-Tier Capstone

Combine concepts from the previous labs into an integrated AWS architecture.

The capstone brings together:

- Networking
- Load balancing
- Compute
- Object storage
- Relational data
- Secure administrative access
- Encryption
- IAM
- Resource tagging
- Cost controls
- HCP Terraform
- Terraform remote state
- Plan, apply, validate, and destroy lifecycle management

The capstone validates actual application connectivity, not simply successful resource creation.

## Optional Serverless Capstone

An optional advanced exercise may introduce a serverless architecture using:

- Amazon CloudFront
- Amazon S3
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB

## Standard Deployment Lifecycle

Infrastructure labs follow this lifecycle:

```text
Review Documentation
        |
        v
Configure Variables
        |
        v
Terraform Plan
        |
        v
Review Plan
        |
        v
Terraform Apply
        |
        v
Validate Infrastructure
        |
        v
Terraform Destroy Plan
        |
        v
Review Destroy Plan
        |
        v
Destroy Infrastructure
        |
        v
Confirm Cleanup
```

HCP Terraform Auto Apply remains disabled so learners review infrastructure changes before they are applied.

## Standard Repository Model

Each infrastructure lab lives in its own repository.

Examples:

```text
terraform-aws-security-training
terraform-aws-network-foundation-lab
terraform-aws-ec2-web-lab
```

This repository provides the curriculum and common standards.

It does **not** deploy AWS infrastructure itself.

## Environment-Specific Information

Public repositories must not contain:

- AWS account IDs
- Internal VPC IDs
- Internal subnet IDs
- AWS access keys
- AWS secret access keys
- HCP Terraform API tokens
- Internal URLs
- Internal organization information
- Terraform state
- Terraform plan files
- Environment-specific `.tfvars` files

Environment-specific values belong in the learner's private HCP Terraform workspace.

## Standard Terraform Variables

Applicable infrastructure labs use common variables such as:

```text
aws_region
expected_aws_account_id
owner_name
deployment_id
expiration_date
```

Lab-specific variables are added where appropriate.

The `expected_aws_account_id` variable provides a guardrail against accidentally deploying training infrastructure into the wrong AWS account.

## Standard Resource Tags

Applicable AWS resources should identify ownership and purpose using tags such as:

```text
ManagedBy
Purpose
Project
Lab
Owner
DeploymentId
Environment
CreatedFrom
ExpiresOn
```

Tags should make it possible to determine:

- Who created a resource
- Which lab created it
- Which deployment owns it
- Whether Terraform manages it
- When it should be removed

## Project Documentation

The standards for the training series are documented in:

- [Training Standards](STANDARDS.md)
- [Security Standard](SECURITY.md)
- [Cost Standard](COSTS.md)
- [AWS Budget Guide](docs/aws-budget-guide.md)
-d-source-control.md
- docs/tagging-and-naming.md
- labs/README.md

## Important Training Notice

These labs are designed for controlled training and sandbox environments.

Example IAM policies provided by the project are training baselines. They should not automatically be treated as production IAM policies.

AWS pricing changes over time and varies by region. Cost estimates must be reviewed against current AWS pricing before public release.

Always review Terraform plans before applying or destroying infrastructure.

## Project Status

This training series is under active development.

Repositories are developed and tested privately before public release while remaining free of private or environment-specific information from the first commit.
