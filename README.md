# SecureMicro

A hands-on delivery lab with a Node.js backend, static frontend, Redis, a Jenkins pipeline, AWS Terraform configuration, and a Kubernetes Helm chart.

[Reviewer walkthrough](INTERVIEW.md) · [Pipeline](Jenkinsfile) · [Infrastructure](infra/main.tf) · [Helm values](helm/myapp/values.yaml)

## Run the application locally

Requires Git, Docker Engine, and Docker Compose v2:

```bash
git clone https://github.com/Vishnu063/SecureMicro.git
cd SecureMicro
docker compose config --quiet
docker compose up --build
```

The Compose file publishes frontend port 8081, backend port 5000, and Redis port 6379. Use a trusted local environment. Stop with Ctrl+C, then run `docker compose down`. The local path is separate from Jenkins and does not require AWS. It has not been freshly executed in this portfolio pass.

## Delivery path in the current source

| Stage | What the Jenkinsfile does |
| --- | --- |
| Checkout and tests | Checks out source and runs the backend npm test command |
| Source scans | Invokes Gitleaks and Trivy filesystem scanning |
| Registry | Authenticates Docker to Amazon ECR using AWS CLI credentials |
| Images | Builds and pushes backend/frontend images, then scans them |
| Deployment | Renders and upgrades the Helm release |
| Inspection | Lists Kubernetes pods and Helm releases |

**Current limitation:** the scan commands use `--exit-code 0`. Findings are advisory; they do not block publication. Images are pushed before their image scans. Listing pods is not a rollout wait or an application health check. The previous README overstated these guarantees.

The Terraform source attaches an EC2 instance profile with the managed ECR PowerUser policy. This is broader than repository-specific least privilege, and the pipeline itself does not prove which AWS identity is used at runtime. AWS account values and image repositories must be configured for the intended account before use.

## What to improve next

- Fail the pipeline on the selected security findings and scan images before push.
- Require an explicit Kubernetes context and namespace, wait for rollout, and verify application traffic.
- Scope registry permissions to the needed repositories.
- Add reproducible monitoring installation and evidence; monitoring setup is not included in the inspected source.
- Record a real failed-build/failed-release demonstration and recovery result.

These are open improvements, not completed features. No Jenkins, AWS, Docker, or Kubernetes execution is claimed by this documentation update.
