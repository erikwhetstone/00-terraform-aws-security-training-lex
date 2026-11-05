# Terraform State and Source Control

Terraform source code and Terraform state serve very different purposes.

Understanding that difference is essential before deploying infrastructure.

## GitHub Owns Source Code

Git stores the Terraform configuration that describes the desired infrastructure.

Examples include:

```text
versions.tf
provider.tf
variables.tf
data.tf
locals.tf
main.tf
checks.tf
outputs.tf
templates/
docs/
```

These files describe what Terraform should do.

## HCP Terraform Owns Shared State

Terraform state records Terraform's understanding of deployed infrastructure.

For this training series, shared state belongs in HCP Terraform.

The basic model is:

```text
GitHub
   |
   | source code
   v
HCP Terraform
   |
   | Terraform operations and state
   v
AWS
```

GitHub is not the state backend.

## What Terraform State Contains

State may include:

- AWS resource IDs
- Public IP addresses
- Private IP addresses
- Generated resource names
- Configuration values
- Resource relationships
- Values supplied to resources
- Potentially sensitive information

State should therefore be treated as operationally sensitive data.

## Files That Must Not Be Committed

Do not commit:

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

Local IDE and operating-system metadata should also normally be excluded:

```text
.vscode/
.idea/
.DS_Store
Thumbs.db
```

## Provider Lock File

The Terraform provider dependency lock file is different.

Commit:

```text
.terraform.lock.hcl
```

The lock file records provider selections and helps produce consistent Terraform initialization behavior.

## Why `.tfvars` Is Ignored in This Training Series

Terraform variable files can contain environment-specific values such as:

- AWS account IDs
- VPC IDs
- Subnet IDs
- CIDR ranges
- Owner identifiers
- Deployment-specific configuration

For this training model, real environment values belong in the learner's private HCP Terraform workspace.

Example files may be committed when they contain placeholders only.

For example:

```text
terraform.tfvars.example
```

must never contain actual credentials or private infrastructure identifiers.

## Git History Matters

Removing a secret from the current version of a file does not necessarily remove it from previous Git commits.

This means a repository should be public-ready from the beginning.

Do not temporarily commit:

- Credentials
- AWS account IDs
- Private VPC or subnet IDs
- Internal URLs
- Private screenshots
- HCP Terraform tokens

with the intention of deleting them later.

## What Happens During Terraform Apply?

Terraform evaluates:

1. The configuration
2. Input variables
3. Existing state
4. Current provider information
5. Current AWS resources

Terraform then calculates the difference between the desired configuration and the observed infrastructure.

After changes are applied, Terraform updates state.

State is therefore part of Terraform's resource ownership model.

## Partial Apply Failures

A failed Terraform run does not necessarily mean that nothing was created.

For example:

```text
Terraform starts
      |
      v
Resource A created
      |
      v
Resource B created
      |
      v
Resource C fails
      |
      v
Run fails
```

Resources A and B may still exist.

Terraform state may also reflect some or all of the successful operations.

Do not immediately delete resources manually after a failed apply.

First review:

- Terraform run output
- HCP Terraform state
- AWS resources
- Any tainted resources
- The underlying error

Correct the issue and let Terraform reconcile ownership whenever practical.

## Why Manual Deletion Can Cause Problems

Terraform state may say:

```text
Resource exists
```

while AWS says:

```text
Resource was manually deleted
```

That creates drift.

Terraform must then reconcile the state with reality during a future plan or refresh.

Manual cleanup is sometimes necessary, but it should be an intentional recovery action rather than the first response to a failed run.

## Data Sources and Ownership

Terraform data sources read existing information.

For example, a later lab may receive a VPC ID from the Network Foundation lab and inspect that VPC using a data source.

Reading that VPC does not make the downstream lab its owner.

This distinction matters during destroy operations.

## Shared Infrastructure

Later labs may depend on resources created by earlier labs.

For example:

```text
Network Foundation
      |
      | VPC and subnet IDs
      v
EC2 Web Lab
```

The EC2 Web Lab may consume network identifiers without creating or owning the network.

Destroying the EC2 workspace should therefore not destroy the shared network foundation.

## Plan Before Apply

Before applying, review the plan for:

- Unexpected resources
- Unexpected deletion
- Resource replacement
- Public access
- IAM changes
- Incorrect AWS account
- Incorrect AWS Region
- Unexpected resource size or cost

A successful Terraform plan is not the same thing as an approved Terraform plan.

Human review is part of the workflow.

## Plan Before Destroy

Destroy operations require review too.

Confirm:

- The correct workspace is selected
- Only lab-owned resources are targeted
- Shared prerequisite infrastructure is not targeted
- Billable resources will actually be removed

Do not treat a destroy confirmation as an automatic step.

## Standard Lifecycle

The training workflow is:

```text
Review
   |
   v
Plan
   |
   v
Review plan
   |
   v
Apply
   |
   v
Validate
   |
   v
Destroy plan
   |
   v
Review destroy plan
   |
   v
Destroy
   |
   v
Confirm cleanup
```

## Source-Control Goal

A public Terraform repository should contain everything needed to understand the architecture without containing the private values needed to target a specific AWS environment.

That separation allows the same source code to be safely reused by different learners.
