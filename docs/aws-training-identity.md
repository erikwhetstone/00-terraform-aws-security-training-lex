# AWS Training Identity Guide

This guide explains how to create the AWS identity that HCP Terraform uses to deploy the training labs.

If you do not administer your AWS account, give this guide to your AWS administrator. They can create the identity and hand you the access key.

## Why a Dedicated Identity

HCP Terraform needs AWS credentials to run plans and applies. Those credentials should belong to a dedicated training identity, not to a person.

A dedicated identity:

- Has only the permissions the labs need.
- Can be disabled without affecting anyone's personal access.
- Makes AWS activity from the labs easy to recognize.

Never use the AWS account root user, a personal administrator login, or the `AdministratorAccess` policy for the labs.

## What You Will Create

| Item | Example name | Purpose |
|---|---|---|
| IAM policy, one per lab | `TerraformTrainingNetworkFoundation` | The permissions one lab needs |
| IAM user | `terraform-training-hcp` | The identity HCP Terraform signs in as |
| Access key | Created on the user | The credentials stored in HCP Terraform |

Each lab repository includes its policy at:

```text
examples/iam/training-policy.json
```

## Step 1: Create the Lab Policy

1. In the IAM console, open **Policies** and select **Create policy**.
2. Choose the **JSON** editor.
3. Replace the contents with the lab's `examples/iam/training-policy.json`.
4. Name the policy after the lab, such as `TerraformTrainingNetworkFoundation`.
5. Create the policy.

With the AWS CLI, from the lab repository folder:

```bash
aws iam create-policy --policy-name TerraformTrainingNetworkFoundation --policy-document file://examples/iam/training-policy.json
```

## Step 2: Create the IAM User

1. In the IAM console, open **Users** and select **Create user**.
2. Name the user, such as `terraform-training-hcp`.
3. Do not give the user AWS console access. It only needs programmatic access.
4. Choose **Attach policies directly** and select the lab policy from Step 1.
5. Create the user.

## Step 3: Create an Access Key

1. Open the user and select the **Security credentials** tab.
2. Under **Access keys**, select **Create access key**.
3. Choose the option for an application running outside AWS.
4. Create the key.

AWS shows the secret access key only once. Store it in a password manager, then enter both values into HCP Terraform as sensitive environment variables. See `hcp-terraform-variables.md`.

Never commit an access key, and never paste one into chat, email, tickets, or screenshots.

## Step 4: Add Policies for Later Labs

Each lab has its own policy. When you start a new lab, create its policy and attach it to the same user.

Labs that build on each other need both policies attached while both are deployed. For example, the EC2 lab deploys into the network from Lab 1, so the user needs the Lab 1 policy and the EC2 lab policy at the same time.

## Testing a Policy on Its Own

Permissions from every attached policy add together. If the user already has other policies, they can hide a permission that a lab policy is missing, and the gap only appears for someone who has that lab policy alone.

When you need to prove that one lab policy is complete, temporarily attach only that policy, run the full lab lifecycle, then reattach the others.

The IAM policy simulator can preview whether an identity is allowed an action:

```bash
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam::ACCOUNT-ID:user/terraform-training-hcp --action-names ec2:CreateVpc --query 'EvaluationResults[].[EvalActionName,EvalDecision]' --output table
```

The simulator shows what a policy permits. It cannot tell you which API calls Terraform will make, so a real plan, apply, and destroy are still the final test.

## Refining a Policy

Add permissions only when a real run fails and names the denied action.

1. Record the exact action in the `AccessDenied` or `UnauthorizedOperation` error.
2. Add only that action to the lab's `examples/iam/training-policy.json`.
3. Publish the updated file as a new version of the IAM policy.
4. Queue a new run.

To publish a new version with the AWS CLI:

```bash
aws iam create-policy-version --policy-arn arn:aws:iam::ACCOUNT-ID:policy/TerraformTrainingNetworkFoundation --policy-document file://examples/iam/training-policy.json --set-as-default
```

For example, testing Lab 1 showed that deleting a subnet requires `ec2:DescribeNetworkInterfaces`, because Terraform checks each subnet for leftover network interfaces first. That action was added only after a destroy failed without it.

An IAM policy keeps at most five versions. Delete old versions you no longer need before publishing a sixth.

## Access Key Hygiene

- Keep at most one active key per purpose. AWS allows two keys per user so you can rotate without downtime.
- Rotate the key periodically: create a new key, update HCP Terraform, then deactivate and delete the old one.
- Deactivate the key when you finish the series.
- If a key is ever exposed, deactivate it immediately and create a new one.

Long-lived access keys are acceptable for the early labs. Lab 8 replaces them with short-lived credentials using HCP Terraform workload identity, which removes the need to store AWS keys at all.
