# DevSecOps CI/CD Pipeline — Jenkins + Trivy + SonarQube + EKS

End-to-end DevSecOps pipeline that builds, scans, and deploys a Java Maven application to Amazon EKS with security checks at every stage.

---

## What this project does

Push code → Jenkins picks it up → scans for vulnerabilities → builds Docker image → scans the image → pushes to ECR → deploys to EKS.

If any security scan fails, the pipeline stops. Nothing gets deployed unless it passes all checks.

---

## Pipeline Stages

```
Stage 1 → Git Checkout
Stage 2 → Trivy FS Scan         (scan source code for CVEs)
Stage 3 → Maven Build + SonarQube  (build + code quality gate)
Stage 4 → ECR Login
Stage 5 → Docker Build
Stage 6 → Trivy Image Scan      (scan container image for CVEs)
Stage 7 → Push to ECR
Stage 8 → Update K8s Manifests  (update image tag)
Stage 9 → Deploy to EKS         (with rollback on failure)
```

---

## Tech Stack

- **Jenkins** — CI server running on EC2
- **Maven** — builds the Java application
- **SonarQube** — code quality and SAST analysis
- **Trivy** — vulnerability scanning (both filesystem and image)
- **Docker** — containerization
- **Amazon ECR** — private Docker registry
- **Amazon EKS** — Kubernetes cluster for deployment
- **RBAC** — least privilege access for Jenkins

---

## Security highlights

**Two Trivy scans:**
- FS scan runs before build — catches dependency CVEs early
- Image scan runs after build — catches OS package CVEs in container

**SonarQube quality gate:**
- Enforces minimum 80% code coverage
- Pipeline fails if gate is not met

**Least privilege access:**
- Jenkins agent uses IAM instance profile — no hardcoded credentials
- aws-auth ConfigMap maps Jenkins IAM role to `system:authenticated` only
- ClusterRole grants only what Jenkins needs — deployments, services, pods, namespaces

**Topology spread:**
- 2 replicas spread across availability zones
- Auto rollback if deployment fails within 60 seconds

---

## Infrastructure

| Component | Details |
|---|---|
| Jenkins Controller | EC2 — Ubuntu 24.04 |
| Jenkins Agent | EC2 — Docker, Maven, Trivy, kubectl installed |
| SonarQube | EC2 — PostgreSQL backend |
| EKS Cluster | Kubernetes 1.32 — 2 nodes across 2 AZs |
| ECR | Private registry — immutable tags enabled |

---

## Project Structure

```
.
├── src/                     ← Java application source
├── pom.xml                  ← Maven config with Jacoco + SonarQube
├── Dockerfile               ← Container image
├── Jenkinsfile              ← Pipeline definition
├── deploy-svc.yaml          ← Kubernetes Deployment + Service
└── clusterrole-binding.yaml ← RBAC manifests
```

---

## How to run

**1. Set up infrastructure**
- Launch Jenkins controller, Jenkins agent, SonarQube on EC2
- Create EKS cluster using eksctl
- Create ECR repository

**2. Configure Jenkins**
- Add GitHub PAT credentials
- Add SonarQube token credentials
- Attach IAM role to Jenkins agent EC2

**3. Configure SonarQube**
- Create project and quality gate
- Generate token for Jenkins

**4. Trigger pipeline**
- Push code to main branch
- Pipeline triggers automatically via GitHub webhook

---


