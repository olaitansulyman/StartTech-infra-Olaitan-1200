# GitHub Actions OIDC Setup

This repository is `olaitansulyman/StartTech-infra-Olaitan-1200`. The Terraform module under `terraform/modules/github-oidc/` creates an AWS OIDC provider and an IAM role for GitHub Actions.

## Before configuring OIDC

1. Review the role trust policy and IAM permissions in `terraform/modules/github-oidc/main.tf`.
2. The current role policy grants broad service permissions with wildcard resources. It is suitable only for a controlled learning environment after review, not for production workloads. Replace it with narrowly scoped permissions before production use.
3. Protect the `main` branch and require review before merging infrastructure changes.
4. Configure the S3 state bucket with encryption, versioning, public-access blocking, and restricted access; configure the DynamoDB lock table.

## Repository identity

Use these Terraform variable values:

```hcl
github_org  = "olaitansulyman"
github_repo = "StartTech-infra-Olaitan-1200"
```

The OIDC trust condition should be limited to the intended repository and branch. Avoid a wildcard subject that permits any branch, tag, or environment to assume the deployment role.

## GitHub Actions secrets

Configure the following repository secrets only if the corresponding workflow requires them:

- `AWS_ACCOUNT_ID` — AWS account ID used to construct the role ARN
- `TFSTATE_BUCKET` — private S3 bucket used for Terraform remote state
- `MONGODB_ATLAS_PUBLIC_KEY`
- `MONGODB_ATLAS_PRIVATE_KEY`
- `MONGODB_ATLAS_ORG_ID`
- `MONGODB_ATLAS_PASSWORD`
- `ASG_NAME` — Auto Scaling Group target for backend deployment
- `FRONTEND_S3_BUCKET` — frontend deployment bucket
- `CLOUDFRONT_DISTRIBUTION_ID` — CloudFront distribution used by the frontend workflow

Do not add AWS access key ID/secret access key pairs. The intended authentication mechanism is OIDC and short-lived AWS credentials.

## Workflow behavior

- Infrastructure validation and planning run on pull requests and pushes to `main`.
- Infrastructure apply is available only through manual workflow dispatch on `main`, with the `apply` input explicitly set to true.
- Frontend/backend deployment workflows should be restricted to `main` and should not deploy from arbitrary branches.

## How OIDC works

1. A GitHub Actions job requests an OIDC token.
2. The AWS credentials action exchanges the token with AWS STS.
3. AWS validates the token's audience and subject against the IAM role trust policy.
4. AWS returns temporary credentials for the role session.
5. The workflow uses those temporary credentials for the permitted actions.

## Production hardening checklist

- [ ] Restrict the OIDC subject to the exact repository and protected branch.
- [ ] Replace wildcard IAM actions/resources with least-privilege permissions.
- [ ] Use a protected GitHub environment with required reviewers for infrastructure apply.
- [ ] Restrict the state bucket and enable versioning/encryption.
- [ ] Review workflow permissions and pin third-party actions to immutable commit SHAs.
- [ ] Scan dependencies and container images; fail builds on defined security thresholds.
- [ ] Confirm Terraform state and plan artifacts cannot expose secrets.
