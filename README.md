# StartTech — AWS Infrastructure and CI/CD Portfolio Project

A hands-on DevOps portfolio project that brings together Terraform-managed AWS infrastructure, GitHub Actions automation, container builds, and basic application observability.

> **Project status:** Learning/portfolio implementation. The repository contains infrastructure code, deployment scripts, workflows, and operational documentation. Do not assume that the stack is currently deployed or serving production traffic; deployment status must be verified independently.

## What this project demonstrates

- **Infrastructure as Code:** Terraform modules for AWS networking, compute, storage/CDN, monitoring, and GitHub Actions OIDC.
- **AWS networking and compute:** VPC, public/private subnets, security groups, an Application Load Balancer, and an EC2 Auto Scaling Group.
- **Frontend delivery:** React build output published to Amazon S3 and distributed through CloudFront.
- **Backend delivery:** Go API container build and Amazon ECR image publishing, with an Auto Scaling refresh workflow.
- **Data/cache integration:** MongoDB Atlas and Amazon ElastiCache configuration.
- **Observability:** CloudWatch log group, dashboard, alarm definitions, and Log Insights queries.
- **Automation and operations:** GitHub Actions workflows, deployment/health-check/rollback scripts, and a runbook.

## Architecture at a glance

```text
                       GitHub Actions
                      /      |       \
              Terraform   Frontend   Backend
                 |          |          |
                 v          v          v
             AWS VPC      S3 + CDN    ECR image
                 |                     |
              ALB + ASG <---------------+
                 |
              Go API
                 |
         MongoDB Atlas / Redis

          CloudWatch logs, metrics,
          dashboards and alarms
```

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the detailed design.

## Repository layout

```text
.
├── .github/workflows/       # Infrastructure, frontend and backend workflows
├── backend/                 # Go API and Dockerfile
├── frontend/                # React application
├── terraform/               # Root configuration and reusable modules
├── monitoring/              # CloudWatch dashboard, alarms and queries
├── scripts/                 # Deploy, health-check, test and rollback helpers
└── docs/                    # Architecture, deployment, OIDC, testing and runbook
```

## Prerequisites

- AWS account and CLI
- Terraform version supported by the root configuration
- Docker
- Go and Node.js versions compatible with the application manifests
- GitHub repository access
- MongoDB Atlas account and API credentials, if using the Atlas resources

## Safe local workflow

1. Review the architecture, variables, IAM policies, and estimated AWS costs before provisioning.
2. Copy the example variables file and provide your own values locally:

   ```bash
   cd terraform
   cp terraform.tfvars.example terraform.tfvars
   ```

3. Never commit `terraform.tfvars`, credentials, private keys, Terraform state, plan files, or generated outputs.
4. Initialize and validate:

   ```bash
   terraform init
   terraform fmt -check -recursive
   terraform validate
   terraform plan
   ```

5. Review the complete plan and expected charges before applying. Run `terraform apply` only in an account you control and only after confirming the plan.
6. Use the scripts and [docs/RUNBOOK.md](docs/RUNBOOK.md) to verify application health and understand rollback steps.
7. Destroy lab resources when finished, after reviewing the destroy plan.

## CI/CD and AWS authentication

The repository includes separate GitHub Actions workflows for infrastructure, frontend, and backend. AWS authentication is designed around GitHub OIDC rather than long-lived AWS access keys.

The infrastructure workflow is intended to run formatting, initialization, validation, and planning automatically. **Applying infrastructure should be an explicit, reviewed action**, not an automatic side effect of every push.

Read [docs/GITHUB-ACTIONS-OIDC.md](docs/GITHUB-ACTIONS-OIDC.md) before configuring AWS trust. Configure only the required GitHub Actions secrets and use a narrowly scoped IAM role. The current Terraform OIDC role policy still needs a full least-privilege review before production use.

## Observability and operations

- CloudWatch dashboard definition: `monitoring/cloudwatch-dashboard.json`
- Alarm definitions: `monitoring/alarm-definitions.json`
- Log Insights examples: `monitoring/log-insights-queries.txt`
- Operational procedures: [docs/RUNBOOK.md](docs/RUNBOOK.md)
- Validation guidance: [docs/TESTING-GUIDE.md](docs/TESTING-GUIDE.md)

## Security and cost notes

- Terraform state can contain sensitive values. Use a private, encrypted, versioned remote backend with state locking.
- Use GitHub OIDC and short-lived credentials; do not store AWS access keys in repository files.
- Review IAM policies for least privilege. The current OIDC role policy is broad and is not production-hardened.
- Keep application secrets in a managed secret store and out of source control.
- NAT Gateways, load balancers, EC2 Auto Scaling, ElastiCache, and CloudFront can incur charges. Review costs and destroy lab resources when they are no longer needed.

## Learning outcomes

This project is intended to demonstrate practical familiarity with Terraform structure, AWS networking and compute, CI/CD workflow design, container image delivery, OIDC-based authentication, CloudWatch observability, and operational documentation. It is a portfolio project, not a claim of several years of production SRE/DevOps operations.
