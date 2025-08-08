# DevOps Project - Strapi Blue/Green Deployment on AWS ECS using Terraform and CodeDeploy

## Overview

This Project sets up a **Blue/Green deployment** strategy for a Strapi CMS application hosted on **AWS ECS Fargate**, using **Terraform** for infrastructure as code and **CodeDeploy** for managed deployment strategies.

## Project Structure

```
.
├── my-strapi-project
    ├── Dockerfile
├──.github
      ├── workflows
            ├── strapi-deploy.yml
            ├── cd.yml    
├── main.tf
├── variables.tf
├── terraform.tfvars
├── outputs.tf
```

---

## Architecture

- **ECS Cluster with Fargate**: Hosts the containerized Strapi app.
- **Application Load Balancer (ALB)**: Distributes incoming traffic between Blue and Green target groups.
- **Two Target Groups (Blue & Green)**: For routing traffic during Blue/Green deployment.
- **Security Groups**: Control inbound/outbound traffic between ALB and ECS.
- **ECS Task Definition**: Container blueprint for running the Strapi app.
- **CodeDeploy Application & Deployment Group**: Orchestrates the Blue/Green deployment strategy.
---
## Why Each Service is Used

### ECS Fargate

- Manages containerized Strapi app without provisioning servers.
- Handles scaling, networking, and orchestration.

### Application Load Balancer (ALB)

- Routes external traffic to ECS services.
- Used for Blue/Green switching during deployment.
- Listens on HTTP (port 80) and HTTPS (port 443).

### Target Groups (Blue & Green)

- ALB uses these to forward traffic.
- During deployment, CodeDeploy swaps traffic between Blue (active) and Green (new).

### ECS Task Definition

- Specifies Docker image, environment variables, port mappings (port 1337), and resource limits.

### Security Groups

- ALB Security Group: Allows HTTP/HTTPS access.
- ECS Security Group: Allows traffic from ALB on port 1337.

### CodeDeploy

- Manages Blue/Green deployment with rollback.
- Uses strategy: `CodeDeployDefault.ECSCanary10Percent5Minutes` or `AllAtOnce`.
- Terminates old tasks post successful deployment.
---
## Blue/Green Deployment Explained

Blue/Green deployment strategy ensures zero-downtime deployment:

- **Blue**: Current production version.
- **Green**: New version being deployed.
- ALB initially routes to Blue.
- During deployment, traffic is shifted to Green gradually (Canary) or instantly (AllAtOnce).
- If something breaks, traffic rolls back to Blue.
---
## Workflow (CI/CD)

1. **CI Pipeline** (GitHub Actions)
   - Builds Docker image.
   - Pushes it to AWS ECR.

2. **CD Pipeline** (GitHub Actions Manual Trigger)
   - Uses the image URI from CI.
   - Runs `terraform apply` to update ECS Task Definition.
   - Triggers CodeDeploy to swap Blue/Green target groups.
---
## Prerequisites

- AWS CLI configured.
- IAM role with permissions for ECS, ALB, CodeDeploy, IAM.
- Docker installed.
- GitHub Secrets set: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `ECR_REPOSITORY`, `AWS_REGION`, `IMAGE_TAG`.
---
## Terraform Files

- `main.tf`: Declares resources (ECS, ALB, CodeDeploy, etc.).
- `variables.tf`: Input variables (e.g., image URI, subnet IDs).
- `outputs.tf`: Outputs like ALB DNS, log group, and alarms.
---
## Validation

- Verify the active target group has a running task.
- Check ALB DNS for Strapi admin panel.
- CodeDeploy console shows success/failure status.
- `terraform output` gives resource info.
---
## Conclusion

This setup ensures **robust, zero-downtime deployments** of your Strapi application using **Blue/Green deployment** via **AWS ECS Fargate, ALB, and CodeDeploy**, all provisioned using **Terraform** and integrated with **GitHub Actions CI/CD pipelines**.

---
## Author

**Rohana Upadhyaya**     
Location: Bengaluru, Karnataka
