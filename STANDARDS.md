# Terraform AWS Security Training Standards

This document defines the common technical and documentation standards for the Terraform AWS Security Training series.

Individual labs may add requirements, but they should not weaken these baseline standards without clearly documenting why.

## Core Principles

Every lab should:

- Teach Terraform concepts rather than only provide deployable code.
- Explain what important AWS resources do and why they exist.
- Use security-conscious defaults.
- Use least-privilege IAM guidance appropriate for training.
- Make AWS cost visible before deployment.
- Include cleanup as part of the exercise.
- Use HCP Terraform for shared remote execution and state.
- Keep credentials and environment-specific values outside source code.
- Begin private development in a public-ready state.
- Remain understandable as a standalone repository.
- Fit into the larger training curriculum.

## Repository Model

Each infrastructure lab should live in its own Git repository.

Examples:

```text
01-terraform-aws-network-foundation-lab
02-terraform-aws-ec2-web-lab
```

The curriculum repository is:

```text
00-terraform-aws-security-training
```

The curriculum repository contains standards and learning guidance. It does not deploy AWS infrastructure.

## Standard Infrastructure Repository Structure

Use the following structure when applicable:

```text
NN-terraform-aws-<lab-name>/
|-- .gitignore
|-- .terraform.lock.hcl
|-- README.md
|-- SECURITY.md
|-- COSTS.md
|-- TROUBLESHOOTING.md
|-- CHANGELOG.md
|-- LICENSE
|-- versions.tf
|-- provider.tf
|-- variables.tf
|-- data.tf
|-- locals.tf
|-- main.tf
|-- outputs.tf
|-- checks.tf
|-- examples/
|   |-- terraform.tfvars.example
|   `-- iam/
|       `-- training-policy.json
|-- docs/
|   |-- architecture.md
|   |-- concepts.md
|   |-- deployment-guide.md
|   `-- cleanup-guide.md
|-- diagrams/
|   `-- architecture.png
`-- templates/
```

Not every lab requires every file or directory, but naming and organization should remain consistent across the series.

## Terraform File Conventions

### `versions.tf`

Defines:

- Required Terraform version
- Required provider versions
- Provider compatibility constraints

This file controls compatibility. It should not define AWS resources.

### `provider.tf`

Defines:

- AWS provider configuration
- AWS region selection
- Standard provider-level default tags

Credentials must never be hardcoded in `provider.tf`.

### `variables.tf`

Defines:

- Learner inputs
- Environment inputs
- Deployment identifiers
- Cost-control inputs
- Optional lab features

Environment-specific values must not be embedded in `main.tf`.

### `data.tf`

Reads information Terraform does not own.

Examples include:

- AWS caller identity
- Availability Zones
- Amazon Machine Images
- Existing VPC information
- Existing subnet information

A data source reads existing information. It does not imply that Terraform owns the resource.

### `locals.tf`

Defines calculated or reusable values such as:

- Naming conventions
- Derived identifiers
- Tag maps
- Repeated expressions

### `main.tf`

Defines the primary infrastructure resources taught by the lab.

The name `main.tf` is an organizational convention. Terraform evaluates all `.tf` files in the working directory as one configuration.

### `checks.tf`

Contains safety and correctness guardrails when applicable.

Examples include:

- Authenticated AWS account matches the expected account
- Selected subnet belongs to the selected VPC
- CIDR ranges do not overlap
- EC2 instance type belongs to an approved low-cost allowlist
- Required expiration information is present

### `outputs.tf`

Publishes values needed to:

- Validate the current lab
- Troubleshoot the deployment
- Feed required information to later labs

Examples include:

- VPC ID
- Public subnet IDs
- Private subnet IDs
- Security group IDs
- Instance IDs
- Application URLs

### `templates/`

Contains rendered scripts or configuration files.

Examples include:

- EC2 user-data scripts
- Web pages
- Application configuration templates

## Comment Standard

Comments should explain **why**, not simply repeat what the Terraform syntax already says.

Avoid comments like:

```hcl
# Create a subnet
resource "aws_subnet" "public" {
```

Prefer comments that explain architectural intent:

```hcl
# Public subnets host Internet-facing resources such as the
# Application Load Balancer introduced in a later lab.
resource "aws_subnet" "public" {
```

## Standard Variables

Every applicable infrastructure lab should support these core inputs:

```text
aws_region
expected_aws_account_id
owner_name
deployment_id
expiration_date
```

### `aws_region`

Identifies the AWS Region used by the deployment.

### `expected_aws_account_id`

Identifies the AWS account that the learner expects to use.

Terraform should compare this value with the authenticated AWS caller identity when practical.

The purpose is to prevent accidental deployment into the wrong AWS account.

### `owner_name`

Identifies the learner or resource owner.

This value is used for resource identification and tagging.

Do not require a personal email address.

### `deployment_id`

Provides a unique identifier for a learner's deployment.

This helps distinguish multiple deployments of the same lab.

### `expiration_date`

Indicates when temporary training infrastructure is expected to be removed.

The value should use a consistent date format:

```text
YYYY-MM-DD
```

## Lab-Specific Variables

Labs may introduce additional inputs when necessary.

Examples include:

```text
vpc_id
public_subnet_id
private_subnet_ids
allowed_http_cidr
instance_type
project_name
enable_custom_domain
domain_name
hosted_zone_id
certificate_arn
```

Do not add configurable inputs merely for flexibility.

Inputs should support a clear learning, security, operational, or cost-control purpose.

## HCP Terraform Variable Separation

HCP Terraform distinguishes between environment variables and Terraform variables.

This distinction should be consistent across the training series.

### Environment Variables

Authentication-related configuration belongs in HCP Terraform environment variables.

During the initial authentication model, examples may include:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_DEFAULT_REGION
```

Credential values must be marked sensitive when supported.

Long-lived AWS keys are a transitional training mechanism. The advanced authentication lab replaces them with workload identity and short-lived credentials.

### Terraform Variables

Deployment configuration belongs in HCP Terraform Terraform variables.

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

The guiding principle is:

> Terraform source describes the architecture. Terraform variables describe the target environment. Provider environment variables authenticate the deployment.

## Standard Tags

Use the AWS provider `default_tags` feature wherever the AWS resource supports tagging.

The standard tag set is:

```text
ManagedBy    = Terraform
Purpose      = Training
Project      = terraform-aws-security-training
Lab          = <lab-name>
Owner        = <learner-identifier>
DeploymentId = <unique-deployment-identifier>
Environment  = Lab
CreatedFrom  = terraform-aws-security-training
ExpiresOn    = YYYY-MM-DD
```

Tags should allow an administrator to determine:

- Who owns the resource
- Which training lab created it
- Which deployment or workspace owns it
- Whether Terraform manages it
- When the resource should be removed

Do not place secrets or sensitive information in tags.

## Naming Standard

Resource names should be:

- Predictable
- Generic
- Environment-independent
- Derived from Terraform variables or locals
- Easy to associate with the correct lab and deployment

Names should not contain:

- Company names
- Internal project names
- Internal domains
- Email addresses
- AWS account IDs
- Private infrastructure identifiers

A typical naming pattern may combine:

```text
project + lab + deployment_id
```

AWS services with globally unique naming requirements may additionally use generic AWS account or region context at deployment time.

## Terraform State Standard

Terraform state is infrastructure data, not source code.

Shared lab state belongs in HCP Terraform.

GitHub owns source code. HCP Terraform owns shared Terraform state.

State files must never be committed to Git.

Terraform state may contain:

- Resource identifiers
- Public IP addresses
- Private IP addresses
- Generated resource names
- Infrastructure relationships
- Configuration values
- Sensitive values supplied to resources

Never commit:

```text
.terraform/
terraform.tfstate
terraform.tfstate.backup
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

Commit the Terraform provider dependency lock file:

```text
.terraform.lock.hcl
```

## Source-Control Standard

New repositories are developed privately but must be public-ready from the first commit.

Never place the following into repository history:

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

Private test values belong in private HCP Terraform workspaces rather than Git.

If sensitive information ever enters Git history, deleting it from the current version of a file is not sufficient. Repository history must be treated as containing the exposed value.

## HCP Terraform Execution Standard

HCP Terraform is the standard shared execution environment for the training series.

Unless a lab explicitly teaches another mode:

- Store shared Terraform state in HCP Terraform.
- Connect the appropriate GitHub repository to its workspace.
- Keep Auto Apply disabled.
- Review the Terraform plan before applying.
- Store authentication information outside Terraform source code.
- Store environment-specific deployment values in private workspace variables.

## Deployment Lifecycle

Infrastructure labs should teach the complete lifecycle:

```text
Review documentation
        |
        v
Configure variables
        |
        v
Terraform plan
        |
        v
Review plan
        |
        v
Terraform apply
        |
        v
Validate infrastructure
        |
        v
Terraform destroy plan
        |
        v
Review destroy plan
        |
        v
Destroy infrastructure
        |
        v
Confirm cleanup
```

Successful creation is not sufficient.

A lab is not complete until the learner understands how to validate and safely remove the infrastructure.

## Plan Review Standard

Before approving an apply, learners should review:

- Resources Terraform will create
- Resources Terraform will modify
- Resources Terraform will destroy
- Unexpected replacements
- Unexpected network exposure
- Unexpected IAM permissions
- Unexpected resource types
- Resource sizes that may increase cost
- Target AWS account
- Target AWS Region

Unexpected changes should be investigated before apply.

## Destroy Standard

Every billable infrastructure lab must include a destroy exercise.

Before confirming destroy:

- Review the destroy plan.
- Confirm only lab-owned resources are targeted.
- Verify shared or prerequisite infrastructure is not unintentionally included.
- Review HCP Terraform state if ownership is unclear.

Do not manually delete resources simply because a Terraform operation partially failed.

Terraform state should be inspected first.

## Cost Standard

The initial learning path targets approximately **$10 or less per learner in AWS list-price usage** when labs are used for short sessions and resources are destroyed according to the instructions.

Every billable lab must document:

- Expected session duration
- Expected session cost
- Estimated monthly cost if forgotten
- Most expensive resource
- Free Tier or credit caveats
- Required cleanup
- Services that may continue billing

Do not describe a lab as free merely because a learner may qualify for Free Tier or promotional credits.

Pricing must be reviewed before public release.

## Cost-Control Defaults

Use cost-conscious defaults whenever they meet the learning objective.

Examples include:

- NAT Gateway disabled by default
- Low-cost EC2 instance allowlists
- Short-duration ALB and RDS exercises
- Immediate destroy after validation
- No unnecessary always-on services
- Manual review before apply

Lab 0 introduces an AWS Budget with:

```text
Suggested budget: $10
First warning:    $5
Second warning:   $8
```

AWS Budget notifications are alerts, not automatic resource shutdown controls.

## Documentation Standard

Each infrastructure lab README should contain:

- Goal
- What You Will Learn
- Architecture
- Security Decisions
- Cost Estimate
- Prerequisites
- Repository Structure
- Terraform File Walkthrough
- HCP Terraform Variables
- IAM Permissions
- Plan
- Apply
- Validate
- Destroy
- Troubleshooting
- What Comes Next

Infrastructure labs should also include, where applicable:

```text
SECURITY.md
COSTS.md
TROUBLESHOOTING.md
docs/architecture.md
docs/concepts.md
docs/deployment-guide.md
docs/cleanup-guide.md
diagrams/architecture.png
examples/iam/training-policy.json
```

The documentation must explain not only **what** to do, but also **why** the architecture and security decisions exist.

## Security Documentation Standard

Every infrastructure lab must answer:

- What identities can deploy the lab?
- What is publicly reachable?
- What remains private?
- Where are credentials stored?
- What data is encrypted?
- What logs are produced?
- What prevents deployment into the wrong AWS account?
- What controls cost and resource size?
- How is the infrastructure destroyed safely?
- What has been simplified specifically for training?

See [SECURITY.md](SECURITY.md) for the series-wide security baseline.

## Public-Release Standard

Before a repository becomes public, it must complete:

1. Technical validation
2. Security review
3. Cost review
4. Plan validation
5. Apply validation
6. Functional validation
7. Destroy validation
8. Source sanitization
9. Git-history sanitization
10. Documentation review

The repository must remain free of private or environment-specific information throughout development rather than relying on cleanup immediately before publication.

The detailed public-release procedure is maintained separately from learner-facing documentation.

## Exceptions

A lab may deviate from these standards when the learning objective requires it.

Any exception should:

1. Be intentional.
2. Be documented.
3. Explain the reason.
4. Explain the security or cost trade-off.
5. Provide the safer recommended approach where appropriate.

Training simplifications must not be silently presented as universal production best practices.
