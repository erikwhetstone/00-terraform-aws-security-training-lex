# AWS Budget Guide

This guide introduces AWS Budgets as an early cost-control measure for the Terraform AWS Security Training series.

## Why Create a Budget?

AWS resources can generate real charges.

A training environment may be inexpensive when used for a few hours but significantly more expensive if resources are accidentally left running.

An AWS Budget provides an early warning when account spending approaches a defined threshold.

A budget is not a substitute for cleanup.

## Training Budget

The suggested baseline for this curriculum is:

- Budget amount: $10
- First warning: $5
- Second warning: $8

The approximately $10 target applies to the initial learning path when labs are completed using short sessions and billable resources are destroyed according to the instructions.

Actual charges may vary.

## Important Limitation

AWS Budget notifications are alerts.

They do not automatically:

- Stop EC2 instances
- Delete resources
- Destroy Terraform infrastructure
- Stop an Application Load Balancer
- Stop Amazon RDS charges
- Remove NAT Gateway
- Prevent additional AWS usage

Learners remain responsible for monitoring and cleaning up AWS resources.

## Before Creating the Budget

You should have:

- Access to an AWS training or sandbox account
- Permission to use AWS Billing and Cost Management
- An email address where you can receive budget notifications
- An understanding that AWS pricing varies by service and Region

Do not place personal email addresses into the Terraform training repositories.

Notification addresses are configured privately in AWS.

## Create the Budget

Sign in to the AWS Management Console using the training or sandbox account.

Open:

AWS Billing and Cost Management

Then open:

Budgets

Choose the option to create a budget.

Create a cost budget for the training account.

## Budget Amount

Use the following starting value:

```text
10 USD
```

This is the curriculum's suggested learner budget.

The purpose is to provide visibility into unexpected spending while completing the labs.

## First Warning

Configure a notification at approximately:

```text
5 USD
```

This represents 50 percent of the suggested training budget.

A warning at this point gives the learner time to investigate unexpected resources before approaching the full budget.

## Second Warning

Configure another notification at approximately:

```text
8 USD
```

This represents 80 percent of the suggested training budget.

An alert at this level should trigger an immediate review of running resources and recent lab activity.

## Notification Address

Use an email address that you actively monitor.

Do not store that email address in:

- Terraform source code
- Terraform variables committed to Git
- Screenshots published with the labs
- Public documentation

Budget notification configuration is specific to the learner's AWS environment.

## Responding to an Alert

If you receive a budget warning:

1. Open AWS Billing and Cost Management.
2. Review current and forecasted spending.
3. Identify which AWS services are generating charges.
4. Review active Terraform labs.
5. Check the relevant HCP Terraform workspaces.
6. Determine which resources are still required.
7. Use Terraform to destroy resources that are no longer needed.
8. Confirm cleanup in AWS.
9. Continue monitoring the budget.

Do not immediately delete Terraform-managed resources manually without checking Terraform state and resource ownership.

## Cost Explorer and Billing Views

AWS billing tools can help identify:

- Which services generated charges
- When spending increased
- Whether unexpected resources remain active
- Which Regions are generating usage

Billing data can take time to appear, so cleanup should not be delayed until a charge becomes visible.

## Free Tier and Credits

Do not assume that a lab is free because an AWS account may have:

- Free Tier benefits
- Promotional credits
- Training credits
- Organization-specific discounts

The curriculum uses list-price estimates so that expected costs remain understandable across different learner environments.

## Resources That Deserve Extra Attention

Some services can produce meaningful charges if forgotten.

Examples later in the curriculum include:

- Application Load Balancers
- Amazon RDS databases
- NAT Gateway
- EC2 storage
- Public IPv4 addresses

NAT Gateway is disabled by default in the Network Foundation lab because its ongoing cost is unnecessary for the initial learning objective.

## Terraform Cleanup Is Still Required

An AWS Budget does not know which infrastructure belongs to a Terraform lab.

Terraform state provides that ownership relationship.

For billable labs, the normal workflow is:

```text
Terraform plan
        |
        v
Terraform apply
        |
        v
Validate the lab
        |
        v
Terraform destroy plan
        |
        v
Review the destroy plan
        |
        v
Terraform destroy
        |
        v
Confirm cleanup
```

Always review the destroy plan before approving it.

## Training Goal

The goal of the budget is not merely to avoid a bill.

The learner should develop the habit of considering cost before infrastructure is created.

For every lab, ask:

- What will this cost during the exercise?
- What will this cost if I forget about it?
- Which resource is the most expensive?
- What must I destroy when I am finished?
- How will I verify cleanup?

Cost awareness is part of infrastructure engineering.
