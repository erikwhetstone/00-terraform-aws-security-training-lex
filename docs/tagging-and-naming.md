# Tagging and Naming Standard

Consistent tags and names help learners and administrators understand who owns an AWS resource, why it exists, and when it should be removed.

## Why Tags Matter

Training environments can quickly accumulate resources.

Without consistent metadata, an administrator may not know:

- Who created a resource
- Which lab created it
- Which deployment owns it
- Whether Terraform manages it
- When it should be removed

Tags provide that operational context.

## Standard Tags

Applicable AWS resources should use the following tags:

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

## Standard Values

The baseline values are conceptually:

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

Values enclosed in angle brackets are placeholders.

Do not copy placeholder characters into values unless the lab specifically instructs you to do so.

## ManagedBy

```text
ManagedBy = Terraform
```

This identifies Terraform as the infrastructure-management system.

It helps distinguish Terraform-managed resources from manually created resources.

## Purpose

```text
Purpose = Training
```

This indicates that the resource belongs to a learning environment rather than a production workload.

## Project

```text
Project = terraform-aws-security-training
```

This associates the resource with the overall training series.

## Lab

The `Lab` tag identifies the specific exercise.

Examples:

```text
network-foundation
ec2-web
secure-s3
systems-manager
alb
private-rds
cloudfront
hcp-aws-oidc
three-tier-capstone
```

Use consistent values within a lab.

## Owner

The `Owner` tag identifies the learner or administrator responsible for the deployment.

Use a simple learner identifier.

Do not require:

- Personal email addresses
- Internal company identifiers
- Sensitive personal information

The purpose is resource ownership, not identity verification.

## DeploymentId

`DeploymentId` distinguishes multiple deployments of the same lab.

Examples:

```text
network-01
ec2-web-01
s3-01
```

The value should be understandable to the learner and consistent throughout the deployment.

## Environment

Training resources use:

```text
Environment = Lab
```

Individual advanced exercises may introduce additional environment concepts later, but the baseline curriculum uses `Lab`.

## CreatedFrom

Use:

```text
CreatedFrom = terraform-aws-security-training
```

This identifies the curriculum that produced the resource.

## ExpiresOn

Temporary training resources should include an expected expiration date.

Format:

```text
YYYY-MM-DD
```

Example:

```text
2030-01-15
```

The date is metadata only.

It does not automatically remove the resource.

Learners must still destroy infrastructure when instructed.

## Terraform Variables

The tags are normally driven by Terraform variables.

Core variables include:

```text
owner_name
deployment_id
expiration_date
```

Lab identity may be defined in Terraform locals when it is fixed for a particular repository.

## Provider Default Tags

The training series should use the AWS provider `default_tags` feature wherever appropriate.

Conceptually:

```hcl
provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      ManagedBy    = "Terraform"
      Purpose      = "Training"
      Project      = "terraform-aws-security-training"
      Lab          = local.lab_name
      Owner        = var.owner_name
      DeploymentId = var.deployment_id
      Environment  = "Lab"
      CreatedFrom  = "terraform-aws-security-training"
      ExpiresOn    = var.expiration_date
    }
  }
}
```

This allows supported AWS resources to inherit the common tags consistently.

## Do Not Put Secrets in Tags

AWS tags are metadata.

Never place the following in tags:

- Passwords
- Access keys
- Secret keys
- API tokens
- Private keys
- Sensitive personal information

Tags should be assumed visible to administrators with access to the AWS account.

## Naming Principles

Resource names should be:

- Predictable
- Generic
- Easy to associate with the lab
- Easy to associate with the deployment
- Independent of internal company information
- Derived from variables or Terraform locals when appropriate

## Suggested Naming Pattern

A useful pattern is:

```text
<project>-<lab>-<deployment>
```

Not every AWS service supports the same naming format or length.

Terraform locals can adapt the common pattern to individual service requirements.

## Avoid Internal Names

Do not use:

- Company names
- Internal project names
- Internal DNS domains
- Personal email addresses
- AWS account IDs copied directly into source
- Private infrastructure identifiers

The public source repository should remain reusable by any learner.

## Globally Unique Names

Some AWS services require globally unique names.

Amazon S3 is a common example.

In those cases, the final deployed name may incorporate generic deployment context such as account or Region information at runtime.

The public Terraform source should still avoid hardcoded environment identifiers.

## Names Versus Tags

Names and tags serve related but different purposes.

A resource name makes the resource recognizable.

Tags provide richer ownership metadata.

Do not rely on a resource name alone when the service supports tags.

## Cleanup

Consistent tags make cleanup easier.

When reviewing AWS resources after a lab, the learner should be able to identify:

```text
Lab
Owner
DeploymentId
ExpiresOn
ManagedBy
```

Those values help distinguish lab-owned resources from unrelated or shared infrastructure.

Terraform state remains the primary ownership mechanism for Terraform-managed infrastructure.

Tags provide additional operational visibility.

## Goal

A resource created by the training series should never look anonymous.

An administrator should be able to determine why the resource exists, which lab created it, who owns the deployment, whether Terraform manages it, and when it is expected to be removed.
