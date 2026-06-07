# Jimag Autos Marketplace

Jimag Autos Marketplace is a cloud-native DevOps and SRE portfolio project built to simulate how a real engineering team designs, deploys, secures, and operates a modern web application on AWS.

The platform includes a Next.js frontend, a NestJS inventory API, PostgreSQL, S3-based vehicle image uploads, GitHub Actions CI/CD, Terraform-managed AWS infrastructure, Amazon EKS, ArgoCD GitOps, External Secrets Operator, IAM Roles for Service Accounts, ingress/TLS, autoscaling, and Prometheus/Grafana observability.

This project is not only a car marketplace application. It is a full-stack cloud platform project focused on infrastructure, delivery automation, security boundaries, observability, and production readiness.

---

## Project Purpose

The purpose of this project is to demonstrate practical DevOps and SRE engineering across the full software delivery lifecycle.

The project covers:

- application development and containerization
- CI/CD automation with GitHub Actions
- container image publishing to Amazon ECR
- infrastructure provisioning with Terraform
- Kubernetes workload deployment on Amazon EKS
- GitOps-based delivery with ArgoCD
- secrets management with AWS Secrets Manager and External Secrets Operator
- least-privilege AWS access using IAM Roles for Service Accounts
- S3 image uploads using presigned URLs
- ingress, DNS, and TLS for secure public access
- health checks, autoscaling, metrics, alerts, and operational runbooks

The goal is to show how application code, infrastructure, Kubernetes delivery, cloud security, and observability work together in a real engineering environment.

---

## High-Level Architecture

### Runtime Traffic Flow

```text
User
  → Route53: dev.jimagcars.com
  → AWS Network Load Balancer
  → NGINX Ingress Controller
  → Next.js frontend
  → internal inventory API service
  → PostgreSQL database
  → S3 bucket for vehicle images
```

The current development environment uses NGINX Ingress with cert-manager and Let’s Encrypt for HTTPS. User traffic enters through Route53 and an AWS Network Load Balancer, then NGINX routes requests to the frontend service inside the EKS cluster.

The frontend communicates with the inventory API through an internal Kubernetes service. The inventory API stores vehicle data in PostgreSQL and stores image metadata linked to objects uploaded to S3.

---

### Deployment Flow

```text
Developer pushes code
  → GitHub Actions CI/CD pipeline runs
  → Docker image is built and scanned
  → Image is pushed to Amazon ECR
  → GitOps image tag is updated
  → ArgoCD detects the Git change
  → ArgoCD syncs Kubernetes manifests
  → EKS rolls out the updated workload
```

Application delivery is handled through CI/CD and GitOps. Application repositories build and publish container images, while the GitOps repository remains the source of truth for Kubernetes deployment state.

This separates application build responsibility from deployment state management and makes environment changes auditable through Git.

---

### Secrets Flow

```text
AWS Secrets Manager
  → External Secrets Operator
  → Kubernetes Secret
  → application pod environment variable
```

Sensitive values are not committed to Git. Database connection information is stored in AWS Secrets Manager and synchronized into Kubernetes using External Secrets Operator.

The inventory service receives required secrets from Kubernetes at runtime. IAM permissions are scoped using IRSA so workloads only receive the AWS access they need.

---

### Image Upload Flow

```text
Inventory API generates a presigned S3 upload URL
  → image uploads directly to S3
  → image key is stored in PostgreSQL
  → frontend renders the vehicle image
```

Vehicle image uploads use a presigned URL pattern. The backend generates a temporary upload URL, the image is uploaded to S3, and the S3 object key is stored in the database.

In the current development environment, the S3 bucket allows public read access for browser image rendering. The production direction is to keep S3 private and serve images through CloudFront with Origin Access Control.

---

## Repository Structure

| Repository | Responsibility |
|---|---|
| `inventory-svc` | Backend inventory API built with NestJS, Prisma, PostgreSQL, S3 upload support, health checks, and Prometheus metrics. |
| `car-website-frontend` | Next.js frontend for browsing vehicle listings and rendering vehicle images. |
| `Jimag-GitOps` | ArgoCD and Kubernetes source of truth for workloads, overlays, ingress, TLS, ExternalSecrets, ServiceMonitors, PrometheusRules, and runbooks. |
| `jimag-infra` | Terraform-managed AWS infrastructure including EKS, RDS, S3, ECR, IAM, IRSA, ingress components, monitoring, and platform add-ons. |
| `shared-contracts` | Shared contracts, API types, or documentation used to reduce drift between frontend and backend expectations. |

---

## Current Development Capabilities

The current development environment demonstrates:

- Amazon EKS workload deployment
- ArgoCD app-of-apps GitOps workflow
- GitHub Actions CI/CD and image promotion
- Amazon ECR container image storage
- Kubernetes readiness, liveness, and startup probes
- Horizontal Pod Autoscaler for backend scaling
- NGINX Ingress with HTTPS using cert-manager and Let’s Encrypt
- Route53 DNS integration
- AWS Secrets Manager integration through External Secrets Operator
- IRSA-based backend access to AWS resources
- S3 image upload and rendering flow
- Prometheus scraping backend application metrics
- Grafana visibility for Kubernetes and application metrics
- Prometheus alert rules for backend availability, pod restarts, and 5xx error rate

---

## Production Direction

The development environment validates the platform design and operational flow. The production direction is to harden the architecture with stronger AWS-native patterns and clearer environment separation.

Planned production improvements include:

- reusable Terraform modules
- separate Terraform roots and state per environment or platform layer
- dedicated production GitOps overlays
- AWS Load Balancer Controller with Application Load Balancer
- ACM-managed TLS certificates
- Route53-managed production DNS
- private S3 bucket for vehicle images
- CloudFront distribution with Origin Access Control
- stricter IAM and IRSA boundaries
- centralized logging
- production-grade alert routing
- runbook-backed incident response
- service-level indicators for availability, latency, and error rate
- production readiness documentation

The development setup is intentionally used to validate the full platform path before moving into a more production-grade architecture.

---

## Engineering Principles

This project follows practical DevOps and SRE principles:

- Git as the source of truth
- infrastructure as code
- repeatable deployments
- environment-specific configuration
- no secrets committed to Git
- least-privilege cloud access
- Kubernetes health checks for services
- automated image promotion
- GitOps-driven deployment state
- metrics-based observability
- runbook-backed alerting
- clear separation between development shortcuts and production patterns

---

## Current Status

Jimag Autos Marketplace is actively evolving from a working development environment into a production-ready cloud platform.

Current focus areas include:

- improving production infrastructure layout
- moving image delivery to private S3 and CloudFront
- refining GitOps environment structure
- expanding observability and alert runbooks
- documenting operational workflows
- strengthening production readiness controls
