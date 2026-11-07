# HCP Terraform Variables Guide

This guide explains how the training series separates authentication information from Terraform deployment configuration.

## The Core Principle

The training series uses this separation:

> Terraform source describes the architecture. Terraform variables describe the target environment. Provider environment variables authenticate the deployment.

Keeping these responsibilities separate makes repositories safer and easier to reuse.

## Three Types of Information

Think about the configuration in three categories.

### 1. Terraform Source Code

Terraform source describes infrastructure.

Examples:

```text
provider.tf
variables.tf
data.tf
locals.tf
main.tf
checks.tf
outputs.tf
```

Source code answers questions such as:

- Which AWS resources should exist?
- How should those resources be configured?
- Which security controls should be enabled?
- Which values should learners provide?

Source code belongs in Git.

### 2. Terraform Variables

Terraform variables describe the learner's target deployment.

Core training variables include:

```text
aws_region
expected_aws_account_id
owner_name
deployment_id
expiration_date
```

Depending on the lab, additional variables may include:

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

Environment-specific values should be supplied privately rather than hardcoded into the repository.

### 3. Environment Variables

Provider authentication information belongs in HCP Terraform environment variables.

During the initial credential-based training model, examples may include:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_DEFAULT_REGION
```

AWS credentials must never be committed to Git.

Secret values should be marked sensitive in HCP Terraform when supported.

## HCP Terraform Variable Types

When creating workspace variables, HCP Terraform distinguishes between:

- Terraform variables
- Environment variables

Choose the correct type deliberately.

## Terraform Variable Example

Suppose a lab defines:

```hcl
variable "expected_aws_account_id" {
  type        = string
  description = "AWS account expected to receive the training resources."
}
```

The corresponding HCP Terraform variable is:

```text
Key:
expected_aws_account_id

Category:
Terraform variable

Value:
the learner's AWS account ID
```

The real account ID belongs in the private workspace.

Do not place a real account ID into the public repository.

## Environment Variable Example

AWS authentication may initially use:

```text
Key:
AWS_ACCESS_KEY_ID

Category:
Environment variable
```

and:

```text
Key:
AWS_SECRET_ACCESS_KEY

Category:
Environment variable
```

The secret access key must be treated as sensitive.

Do not define AWS credentials as ordinary Terraform variables merely so Terraform can pass them into the provider.

The HCP Terraform **Add variable** form defaults to **Terraform variable**. When adding AWS credentials, select **Environment variable** before saving. A credential saved as a Terraform variable is not seen by the AWS provider, and the plan fails with `No valid credential sources found`.

## Why This Separation Matters

Consider two concepts:

```text
Authentication:
Who is Terraform?

Deployment configuration:
Where and how should Terraform deploy?
```

AWS credentials answer the first question.

Variables such as `aws_region`, `vpc_id`, and `deployment_id` answer the second.

Mixing these responsibilities makes configurations harder to secure and understand.

## Expected AWS Account Guardrail

Applicable labs use:

```text
expected_aws_account_id
```

Terraform can compare that value against the identity returned by AWS.

Conceptually:

```text
Expected account
      |
      v
Compare
      ^
      |
Authenticated AWS account
```

If the accounts do not match, the AWS provider setting `allowed_account_ids` stops the run before any AWS change. A `check` block alone only produces a warning, so it is not used as the enforcing control.

This is designed to reduce the risk of accidentally deploying training infrastructure into the wrong AWS account.

## Owner Name

The variable:

```text
owner_name
```

identifies the learner or resource owner.

Use a generic identifier appropriate for resource tagging.

A personal email address is not required.

## Deployment ID

The variable:

```text
deployment_id
```

distinguishes one deployment from another.

For example:

```text
network-lab-01
```

or:

```text
ec2-web-01
```

The identifier should help a learner recognize which resources belong together.

## Expiration Date

Temporary training infrastructure should use:

```text
expiration_date
```

The standard format is:

```text
YYYY-MM-DD
```

This value helps administrators understand when temporary infrastructure is expected to be removed.

An expiration tag does not automatically destroy infrastructure.

The learner must still complete the cleanup process.

## Sensitive Does Not Mean Absent From State

Terraform supports sensitive variables and outputs.

Marking something as sensitive can prevent ordinary display in Terraform output.

It does not automatically mean that the value cannot exist in Terraform state.

That is one reason Terraform state must be protected.

## Do Not Commit Workspace Values

Do not copy private HCP Terraform workspace values into:

```text
terraform.tfvars
*.tfvars
*.tfvars.json
README.md
screenshots
example files containing real values
```

Example files may contain obvious placeholders but never real environment values.

## Authentication Evolution

The initial labs may use static AWS credentials so learners can understand the relationship between:

- GitHub
- HCP Terraform
- AWS IAM
- Terraform providers

This is a transitional model.

A later lab introduces HCP Terraform workload identity with AWS OpenID Connect and AWS STS.

That advanced model replaces long-lived AWS access keys with short-lived credentials.

## Summary

Use HCP Terraform variables according to purpose:

```text
Terraform source
    = architecture

Terraform variables
    = deployment configuration

Environment variables
    = provider authentication
```

Never store real credentials or private environment-specific values in the public training repositories.
