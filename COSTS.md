# Terraform AWS Security Training Cost Standard

This document defines the cost-management standard for the Terraform AWS Security Training series.

## Whole-Series Target

The initial learning path targets approximately $10 or less in AWS list-price usage per learner.

This target assumes:

- Normal low training traffic
- Short lab durations
- Prompt cleanup after validation
- NAT Gateway disabled by default
- No unnecessary always-on resources
- Billable labs are treated as temporary training environments

The $10 target is a design goal, not a billing guarantee.

AWS pricing varies by service, region, usage, and time. Free Tier benefits, promotional credits, or organization-specific discounts must not be assumed when publishing cost estimates.

Final lab cost estimates must be reviewed against current AWS pricing before public release.

## Cost Is Part of the Lab

Cost awareness is a required learning objective.

Learners should understand:

- Which resources cost money
- How AWS measures that usage
- Which resource is likely to be the most expensive
- What happens if resources are forgotten
- Which resources continue billing when stopped
- How to verify cleanup after a Terraform destroy

A successful deployment is not complete until the learner understands the cost and cleanup implications.
## Required Cost Information for Every Lab

Every infrastructure lab must document:

- Expected session duration
- Expected session cost
- Estimated monthly cost if resources are forgotten
- Most expensive resource in the lab
- Free Tier or promotional credit caveats
- Required cleanup actions
- Services that may continue billing when stopped

Cost estimates should use AWS list prices rather than assuming that a learner receives Free Tier benefits, credits, or organization-specific discounts.

## AWS Budget Baseline

Lab 0 introduces AWS Budgets as an early cost-control measure.

The suggested learner baseline is:

- Training budget: $10
- First warning: $5
- Second warning: $8

AWS Budget notifications are alerts. They do not automatically stop, terminate, or destroy AWS resources.

Learners remain responsible for monitoring resources and completing cleanup.

## Cost-Control Defaults

The training series uses cost-conscious defaults whenever they satisfy the learning objective.

Examples include:

- NAT Gateway disabled by default
- HCP Terraform Auto Apply disabled
- Small EC2 instance types
- Short-duration ALB and RDS exercises
- No unnecessary always-on resources
- Prompt Terraform destroy after validation
- Destroy exercises for every billable lab

NAT Gateway requires particular care because its ongoing hourly and data-processing charges can materially increase the cost of a training environment.
## Approximate Cost Categories

The following categories provide planning guidance for the curriculum:

- VPC, subnets, route tables, and Internet Gateway: normally $0 when idle
- Small S3 training content: typically pennies for light use
- Small EC2 training sessions: typically pennies to low cents, plus applicable IPv4 and storage charges
- CloudFront with private S3 and light traffic: generally low cost
- Application Load Balancer: use only for short-duration labs
- Amazon RDS: use only for short-duration labs
- NAT Gateway: avoid by default because ongoing charges can materially increase training cost
- Traditional three-tier capstone: target approximately $0.50 to $1.00 for a limited session

These are curriculum planning estimates, not pricing guarantees.

## Forgotten Resources

Stopping a resource does not necessarily eliminate all charges.

Depending on the lab, costs may continue for items such as:

- Storage
- Public IPv4 addresses
- Load balancers
- Databases
- NAT Gateway
- Stored objects
- Snapshots or backups

Each lab must explain which resources continue generating charges and how to remove them safely.

## Release-Time Pricing Review

AWS pricing changes over time and can vary by Region.

Before a lab is publicly released:

1. Review its expected architecture.
2. Identify every billable AWS service.
3. Verify current list pricing using official AWS pricing information or the AWS Pricing Calculator.
4. Update the expected session cost.
5. Update the forgotten monthly cost.
6. Confirm that the whole-series approximately $10 target remains realistic.
7. Confirm that cleanup instructions remove billable lab-owned resources.

Do not describe a lab as free solely because Free Tier benefits or promotional credits may apply.

Cost estimates should be reviewed again whenever the architecture changes materially.
