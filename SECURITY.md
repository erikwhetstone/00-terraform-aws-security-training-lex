# Terraform AWS Security Training Security Standard

This document defines the security baseline for the Terraform AWS Security Training series.

The goal is to teach infrastructure as code using security-conscious defaults while clearly identifying where a training environment differs from a production environment.

## Security Philosophy

Security is part of every lab rather than a separate topic added at the end.

Each lab should teach learners to consider:

- Identity
- Authentication
- Authorization
- Network exposure
- Encryption
- Secrets
- Terraform state
- Resource ownership
- Deployment guardrails
- Logging
- Cost controls
- Cleanup
- Infrastructure lifecycle

Training simplifications must be explicitly documented and must not be presented as universal production best practices.

## Shared Responsibility

Using Terraform does not transfer responsibility for AWS security to Terraform.

The learner remains responsible for understanding:

- Which AWS account is targeted
- Which identity Terraform is using
- Which permissions that identity has
- Which resources will become publicly reachable
- Which values may enter Terraform state
- Which resources generate cost
- Which resources Terraform owns
- Which resources must be destroyed after the lab

Terraform automates infrastructure changes. Terraform does not replace security review.

## Identity and Authentication

Terraform must authenticate to AWS using credentials supplied outside Terraform source code.

Credentials must never be hardcoded into:

- `provider.tf`
- `variables.tf`
- `terraform.tfvars`
- README files
- Shell scripts committed to Git
- Example configuration
- Screenshots
- Git history

The initial training workflow may use AWS access keys stored as sensitive HCP Terraform environment variables.

The advanced authentication lab replaces long-lived AWS credentials with workload identity, AWS IAM trust, and temporary AWS STS credentials.

Long-lived AWS credentials are therefore a transitional learning mechanism, not the desired end-state architecture.

## HCP Terraform Credential Handling

Authentication-related values belong in HCP Terraform environment variables.

Examples during the initial authentication model include:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_DEFAULT_REGION
```

Secret values should be marked sensitive where supported.

Deployment configuration belongs in Terraform variables rather than authentication environment variables.

Examples include:

```text
expected_aws_account_id
owner_name
deployment_id
expiration_date
vpc_id
subnet_id
allowed_http_cidr
```

The separation is intentional:

> Provider environment variables authenticate the deployment. Terraform variables describe the target environment. Terraform source describes the architecture.

## Least Privilege

Recommended training IAM policies must not use `AdministratorAccess`.

IAM permissions should be limited to the services and operations required by the specific lab.

Learners should understand that Terraform often requires both:

- Write permissions to create, modify, and destroy resources
- Read permissions to refresh state and inspect related AWS configuration

A policy that can create a resource may still fail during Terraform refresh if required read permissions are missing.

IAM examples in this project are **training baselines**.

They are not universal production IAM policies.

Production IAM design may require additional controls such as:

- Permission boundaries
- Service Control Policies
- Session policies
- Conditional access
- Workload identity
- Resource-level restrictions
- Organization-specific governance

## AWS Account Guardrail

Applicable infrastructure labs should require:

```text
expected_aws_account_id
```

Terraform should compare the learner-provided expected account ID with the authenticated AWS account when practical.

The deployment should fail when the values do not match.

This guardrail helps prevent accidental creation of training resources in the wrong AWS account.

The expected account ID is environment-specific and must not be hardcoded into a public repository.

## AWS Region

The deployment region must be explicit.

Applicable labs should use:

```text
aws_region
```

Learners should confirm the region during plan review because:

- Service availability may vary
- AWS pricing may vary
- Existing prerequisite infrastructure is regional
- Deploying into the wrong region can make resources difficult to locate and clean up

## HCP Terraform Auto Apply

HCP Terraform Auto Apply should remain disabled for the training series.

Learners must review Terraform plans before infrastructure changes are applied.

Manual review is a security control as well as a learning objective.

## Terraform Plan Review

Before applying a Terraform plan, review at minimum:

- Target AWS account
- Target AWS Region
- Resources being created
- Resources being modified
- Resources being destroyed
- Unexpected resource replacement
- IAM changes
- Security group rules
- Public IP assignment
- Publicly accessible services
- Encryption configuration
- Resource sizes
- Expensive resource types
- Changes to shared prerequisite infrastructure

Do not approve an unexpected plan simply because Terraform generated it successfully.

## Destroy Plan Review

Destroy operations require the same level of attention as creation.

Before approving a destroy operation:

- Review all resources Terraform intends to destroy.
- Confirm the resources belong to the current lab.
- Confirm shared prerequisite resources are not targeted.
- Confirm the correct HCP Terraform workspace is active.
- Inspect Terraform state when ownership is unclear.

A destroy plan should never be treated as a routine confirmation prompt.

## Terraform State Security

Terraform state may contain operationally sensitive information.

Examples include:

- Resource IDs
- Public IP addresses
- Private IP addresses
- Generated names
- Infrastructure relationships
- Configuration values
- Values supplied to resources
- Potentially sensitive application data

Terraform state must never be committed to Git.

Shared training state belongs in HCP Terraform.

The repository `.gitignore` must exclude:

```text
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
tfplan
*.tfvars
*.tfvars.json
.terraformrc
terraform.rc
crash.log
crash.*.log
```

The Terraform provider dependency lock file should be committed:

```text
.terraform.lock.hcl
```

## Partial Terraform Failures

A failed Terraform apply does not necessarily mean that AWS resources were not created.

Terraform may successfully create some infrastructure before encountering an error.

A partial failure can therefore leave:

- Real AWS resources
- Terraform state entries
- Tainted resources
- Billable infrastructure

Do not immediately delete resources manually after a partial Terraform failure.

First:

1. Inspect the Terraform run.
2. Inspect HCP Terraform state.
3. Determine which resources Terraform believes it owns.
4. Correct the configuration or permissions.
5. Reconcile state where necessary.
6. Use Terraform to destroy resources whenever practical.

Manual deletion without understanding state can create additional drift.

## Source-Control Security

New repositories begin as private but must remain public-ready from the first commit.

Never place these values into Git history:

- Company names or internal branding
- Internal email addresses
- Internal AWS account IDs
- Internal VPC IDs
- Internal subnet IDs
- Internal URLs
- Internal screenshots
- AWS access keys
- AWS secret access keys
- HCP Terraform API tokens
- Terraform state
- Terraform plan files
- Environment-specific `.tfvars` files
- Private keys

Public safety should be designed into the repository rather than performed as a cleanup exercise immediately before publication.

## Sensitive Information in Git History

Deleting a secret from the current version of a file does not remove the secret from earlier Git commits.

If sensitive or internal information enters repository history, assume the history contains that information.

For a repository intended for public training, creating a new clean public repository may be safer than assuming a deleted value has been completely removed.

Compromised credentials must also be revoked or rotated. Removing credentials from Git does not make exposed credentials safe again.

## Network Security

Labs should expose only the network paths required for the learning objective.

Avoid broad inbound rules when a more restrictive rule can satisfy the exercise.

Security group rules should make the communication path understandable.

Examples:

```text
Internet
   |
   v
Application Load Balancer
   |
   v
Application Security Group
   |
   v
Private Database
```

Later labs should demonstrate security-group-to-security-group relationships rather than opening backend services broadly to the Internet.

## SSH

SSH is disabled by default in the training series.

EC2 labs should not automatically expose TCP port 22.

Administrative access should progress toward AWS Systems Manager Session Manager.

This teaches learners that administrative access does not inherently require an Internet-exposed SSH endpoint.

A lab that intentionally enables SSH must explain:

- Why SSH is required
- Which source CIDR is allowed
- How credentials or keys are protected
- What safer alternative normally applies

## EC2 Security Baseline

Applicable EC2 labs should use:

- Amazon Linux 2023
- IMDSv2
- Encrypted EBS storage
- Restricted security groups
- Low-cost instance-type allowlists
- Standard resource tags
- No SSH by default

### IMDSv2

EC2 Instance Metadata Service Version 2 should be required.

The configuration should prevent optional fallback to IMDSv1.

Learners should understand that instance metadata can expose powerful temporary credentials when an EC2 instance uses an IAM role.

### EBS Encryption

EC2 storage should be encrypted.

Training examples should explicitly configure encryption where doing so makes the control visible and understandable.

## HTTP and HTTPS

Early labs may use HTTP where the purpose is basic EC2 or networking education.

HTTP exposure should be limited by `allowed_http_cidr` when appropriate.

Later labs introduce:

- Application Load Balancers
- HTTPS
- AWS Certificate Manager
- Amazon CloudFront

A purchased custom domain must not be required for the core curriculum.

AWS-generated hostnames can be used for the basic learning path.

## Amazon S3 Security Baseline

S3 training buckets should be private by default.

Applicable labs should enable or preserve:

- S3 Block Public Access
- Encryption
- Versioning
- Appropriate lifecycle rules

A public S3 bucket should not be required for the standard curriculum.

CloudFront labs should access private S3 content using Origin Access Control rather than making the bucket public.

## Amazon RDS Security Baseline

RDS databases must not be publicly accessible in the standard training architecture.

The database should use:

```text
publicly_accessible = false
```

Applicable RDS labs should use:

- Private or database subnets
- Encryption
- Backup settings
- A database security group
- Application-to-database security-group relationships

The database security group should permit the required database protocol from the application tier rather than from broad Internet CIDR ranges.

## Encryption

Applicable training infrastructure should use encryption for:

- EBS volumes
- RDS storage
- S3 objects

Encryption controls should be visible in the Terraform configuration so learners recognize them as deliberate architectural choices.

Individual labs should document whether AWS-managed keys or customer-managed KMS keys are being used and why.

## Secrets

Secrets must not be stored in Terraform source code, committed variable files, documentation, scripts, screenshots, or Git history.

Sensitive authentication values belong outside source control. HCP Terraform environment variables may be used for the initial training authentication model, with workload identity and short-lived AWS credentials introduced later in the curriculum.

Terraform state must also be treated as sensitive because values handled by Terraform may be represented in state.

## Resource Ownership and Shared Infrastructure

Tags, Terraform state, workspace boundaries, variables, and outputs should make resource ownership clear.

A downstream lab may consume infrastructure created by a prerequisite lab without owning that infrastructure.

Before destroy operations, learners must confirm that shared prerequisite resources are not unintentionally targeted.

## Cost and Security

Cost control is part of the security and governance model.

Labs should use low-cost defaults, restrict unnecessarily large resources, avoid NAT Gateway by default, use expiration tags, introduce AWS Budgets, and require cleanup of billable infrastructure.

See [COSTS.md](COSTS.md) for the complete cost standard.

## Required Security Review

Every infrastructure lab must answer:

1. What identities can deploy the lab?
2. What is publicly reachable?
3. What remains private?
4. Where are credentials stored?
5. What data is encrypted?
6. What logging is available?
7. What prevents deployment into the wrong AWS account?
8. What controls resource size and cost?
9. How is infrastructure destroyed safely?
10. What was simplified specifically for training?

Functional success alone is not sufficient.

Security validation should confirm applicable controls such as restricted network access, IMDSv2, encryption, S3 Block Public Access, private RDS access, correct account and Region, and preservation of shared infrastructure.

## Public Release Security Review

Before public release, verify that the repository and its Git history contain no:

- Internal branding
- Internal email addresses
- Internal AWS account IDs
- Internal VPC or subnet IDs
- AWS credentials
- HCP Terraform tokens
- Private keys
- Terraform state
- Terraform plan files
- Private URLs
- Internal screenshots

IAM examples must remain generic and clearly identified as training baselines.

Repository maintainers should complete a security, source, Git-history, cost, documentation, and lifecycle review before public release.

## Summary

The recurring security principles for this training series are:

- Least privilege
- Explicit authentication
- AWS account verification
- Restricted network exposure
- Encryption
- Terraform state protection
- Secret protection
- Plan review
- Destroy review
- Clear resource ownership
- Cost control
- Safe cleanup
- Short-lived credentials as the advanced authentication target

The goal is not merely to create functioning AWS infrastructure.

The goal is to teach learners how to build, understand, review, validate, and safely remove infrastructure.
