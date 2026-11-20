# StartTech — Setup and Deployment Guide

This repository contains the infrastructure and application code for the StartTech portfolio project. It uses one GitHub repository: `olaitansulyman/StartTech-infra-Olaitan-1200`.

> This is a learning/portfolio environment. Review costs and security before creating resources. Do not treat a successful Terraform apply as evidence of a production-ready service.

## 1. Prerequisites

- AWS account and AWS CLI
- Terraform compatible with the `required_version` in `terraform/main.tf`
- Docker
- Go and Node.js versions compatible with `backend/go.mod` and `frontend/package.json`
- GitHub access to this repository
- MongoDB Atlas organization/API credentials if you intend to provision Atlas resources

## 2. Configure Terraform variables

From the repository root:

```bash
cd terraform
cp terraform.tfvars.example terraform.tfvars
```

Edit `terraform.tfvars` with values for your own AWS and MongoDB Atlas environments. Do not commit this file. The Atlas public/private keys and database password are sensitive inputs.

For a safer setup, provide secrets through a local secret manager or environment variables rather than placing them in shell history. Terraform state may retain sensitive input values even when variables are marked `sensitive`.

## 3. Configure the remote state backend

The root Terraform configuration uses an S3 backend and a DynamoDB lock table. Create the state bucket and lock table in your account before running `terraform init`.

Recommended controls:
- S3 bucket versioning enabled
- Server-side encryption enabled
- Block all public access
- Bucket policy restricted to the Terraform operator/role
- DynamoDB state-lock table
- State access logged and reviewed

The backend bucket name is supplied at initialization:

```bash
terraform init -backend-config="bucket=YOUR_PRIVATE_STATE_BUCKET" -input=false
```

Never commit local or remote state exports, plan files, or Terraform output files.

## 4. Validate and plan before applying

```bash
terraform fmt -check -recursive
terraform validate
terraform plan
```

Review all planned resources, IAM permissions, network exposure, and estimated charges. Apply only after reviewing the plan:

```bash
terraform apply
```

## 5. GitHub Actions with AWS OIDC

The workflows live in `.github/workflows/`. Follow [GITHUB-ACTIONS-OIDC.md](docs/GITHUB-ACTIONS-OIDC.md) to configure the OIDC provider and role trust for this exact repository.

Do not create an IAM user with broad administrator permissions or store long-lived AWS access keys as GitHub secrets. The current Terraform-created OIDC role policy is intentionally called out for a least-privilege review before production use.

The infrastructure workflow validates and plans on code changes but does not apply infrastructure. Review the plan in GitHub Actions, then run `terraform apply` manually from a trusted workstation only after checking the resources and costs. Configure GitHub branch protection and required reviewers in repository settings.

## 6. Application workflows

- Backend workflow: runs Go tests, builds a Docker image, scans it, publishes to ECR, and initiates an Auto Scaling instance refresh when deployment prerequisites are configured.
- Frontend workflow: installs dependencies, builds/tests the React app, scans the image reference, publishes static build output to S3, and invalidates CloudFront.

Before enabling deployments, verify that:
- The AWS OIDC role trust policy names this repository and intended branch.
- The role has only the permissions needed for the workflow.
- Required GitHub secrets/variables are configured.
- The target ECR repository, S3 bucket, CloudFront distribution, and Auto Scaling Group exist.
- Application runtime configuration and database connectivity are valid.

## 7. Verify and operate

Use the scripts in `scripts/` and consult:
- [Architecture](docs/ARCHITECTURE.md)
- [Deployment guide](docs/DEPLOYMENT-GUIDE.md)
- [Operations runbook](docs/RUNBOOK.md)
- [Testing guide](docs/TESTING-GUIDE.md)

Verify the frontend URL, backend health endpoint, target health, CloudWatch logs, and alarms after deployment. Record actual test evidence rather than assuming a workflow definition proves a successful deployment.

## 8. Cleanup

When the lab is no longer required, review and execute the application cleanup and Terraform destroy procedures. Confirm that no billable AWS resources remain. Retain or securely archive Terraform state as required; do not publish it.
