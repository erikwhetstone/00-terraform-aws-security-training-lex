# HCP Terraform Setup Guide

This guide prepares HCP Terraform for the training series. Each lab's deployment guide then walks through creating that lab's workspace.

## What You Need

- An HCP Terraform account. The free tier is enough for this series.
- A GitHub account with your own copy of each lab repository, created by forking it or using it as a template.
- The AWS training identity and access key from `aws-training-identity.md`.

## Step 1: Create an Account and Organization

1. Sign up for HCP Terraform.
2. Create an organization when prompted. Organization names must be unique across HCP Terraform.
3. Optionally, create a project such as `AWS Terraform Training` to group the lab workspaces together.

## Step 2: Connect GitHub

HCP Terraform reads each lab's code from GitHub through the Terraform Cloud GitHub App. The first time you create a workspace with the **Version Control** workflow, HCP Terraform asks you to install it.

When installing the app, choose **Only select repositories** and select only your lab repositories. Granting access to every repository in your account is not necessary.

When you add a lab repository later, the app does not see it automatically. Grant access in GitHub:

1. Open **Settings > Applications > Installed GitHub Apps**.
2. Select **Configure** next to the Terraform Cloud app.
3. Add the new repository under **Repository access** and save.

If a repository does not appear in the HCP Terraform repository list, this is the usual cause.

## Step 3: One Workspace per Lab

Give each lab its own workspace. Each workspace keeps its own state, variables, and run history, so destroying one lab cannot affect another.

Recommended settings for every lab workspace:

| Setting | Value | Why |
|---|---|---|
| Workflow | Version Control | Runs use the code in your repository |
| Execution mode | Remote | Plans and applies run in HCP Terraform |
| Auto-apply | Off | Every plan waits for your review |
| Terraform version | A specific release | Keeps runs reproducible |

## Step 4: Add Variables Carefully

Each lab lists the variables its workspace needs. AWS credentials are **Environment variables**, and deployment settings are **Terraform variables**. See `hcp-terraform-variables.md` for the full explanation.

The **Add variable** form defaults to **Terraform variable**. Check the category every time you add an AWS credential.

## Runs Started by Git Pushes

A VCS-connected workspace queues a plan every time you push to the repository. When the lab is deployed, that plan shows the effect of your change.

When the lab is destroyed, a push queues a plan that recreates the whole lab. Discard that run unless you intend to deploy again.

## Local Validation Is Not Enough

`terraform validate` checks syntax and structure, but some errors only appear during a real plan. For example, calling a function that does not exist inside a `check` block passes `validate` and fails at plan time. Treat the first HCP Terraform plan as part of testing any change.

## Destroying a Lab

Destroy a lab from its workspace under **Settings > Destruction and Deletion** by selecting **Queue destroy plan**. Review the destroy plan before confirming it, and follow that lab's cleanup guide afterward.
