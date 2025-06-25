# AWS-CFT-TF-EKS-RDS

## Overview

This project demonstrates a multi-region infrastructure and deployment setup on AWS using two Infrastructure as Code (IaC) tools:

- **AWS CloudFormation (CFT)** for infrastructure provisioning in **one AWS region**.
- **Terraform (TF)** for infrastructure provisioning in **another AWS region**.

In both regions, the project provisions an **EKS cluster** to deploy a **3-tier application** backed by an **RDS database**. The infrastructure is customized for each region using the respective IaC tool.

---

## Architecture

### Region 1: CloudFormation-based Infra & CodePipelines

- **Infrastructure provisioning**: AWS CloudFormation templates create:
  - Custom VPC and networking resources
  - Amazon EKS cluster
  - RDS database instance
- **AWS CodePipeline workflows**:
  - **Pipeline 1**: Creates/updates the CloudFormation stack
  - **Pipeline 2**: Runs SonarQube scans on the application code, builds Docker images, and deploys the 3-tier app to EKS

---

### Region 2: Terraform-based Infra & CodePipelines

- **Infrastructure provisioning**: Terraform code provisions similar resources as in Region 1:
  - Custom VPC and networking
  - EKS cluster
  - RDS database
- **AWS CodePipeline workflows**:
  - Pipeline to apply Terraform configurations
  - Pipeline to perform SonarQube scans, build and push Docker images, and deploy the 3-tier app to the Terraform-provisioned EKS cluster

---

## 3-Tier Application

The deployed application consists of:

- **Frontend**: React single-page application (SPA) hosted on EKS
- **Backend**: Node.js/Express API server running in EKS
- **Database**: Amazon RDS MySQL instance

---

## SonarQube Integration

- SonarQube scans are integrated into the CodePipelines to ensure code quality.
- The scanner runs during the build phase and blocks deployment on failing quality gates.

---

## Setup Instructions

### Prerequisites

- AWS CLI configured with appropriate permissions
- AWS CodePipeline and CodeBuild configured in both regions
- Docker installed for building images locally if needed
- Access to SonarQube server with authentication token
- Git installed

---

### Deploying in Region 1 (CloudFormation)

1. Run the **CloudFormation stack creation CodePipeline** to provision infrastructure.
2. Run the **application CodePipeline** which:
   - Executes SonarQube scans
   - Builds Docker images
   - Pushes images to ECR
   - Deploys app manifests to EKS

---

### Deploying in Region 2 (Terraform)

1. Apply Terraform configurations either manually or via the Terraform CodePipeline.
2. Run the **application CodePipeline** (similar to Region 1) for SonarQube scan, build, and deploy.

---

### Accessing the Application

- The Kubernetes manifests include an **Ingress** resource.
- This Ingress provisions an AWS Load Balancer that exposes the frontend and backend services externally.
- You can access the application via the Load Balancer's DNS name provided by the Ingress.

---

## Route 53 Failover Routing Configuration

To provide high availability and automatic failover between the two regions (CloudFormation-based and Terraform-based), this project uses **Route 53 failover routing** for the application domain:

- **Domain:** `studentteacher.threetierashutoshproject1197.xyz`
- **Failover Configuration:**
  - Two Route 53 A (or Alias) records are created under this domain, each pointing to the AWS Load Balancers provisioned by the two infrastructure setups:
    - **Primary Record:** Points to the CloudFormation provisioned Load Balancer in Region 1.
    - **Secondary (Failover) Record:** Points to the Terraform provisioned Load Balancer in Region 2.
  - Health checks are configured on the primary load balancer's endpoint to detect availability.
  - If the health check fails, Route 53 automatically fails over DNS resolution to the secondary load balancer.
  
### Benefits
- Users are automatically routed to the healthy application endpoint without manual intervention.
- Provides multi-region resilience and high availability.

### Implementation Notes
- Ensure both load balancers have DNS names registered as Alias targets in Route 53.
- Create appropriate health checks targeting application endpoints (e.g., `/health` path on the frontend or backend).
- Update DNS TTL (time-to-live) to a low value (e.g., 60 seconds) for quicker failover response.
- Configure your SSL certificates accordingly for the domain on both load balancers if using HTTPS.

## Notes
- Ensure CodePipeline environments have required IAM roles and permissions.
- Kubernetes manifests use image tags generated dynamically during builds.
- Secrets like DB credentials are stored securely using Kubernetes Secrets.
- Adjust region-specific parameters and secrets in the pipeline environment variables.

---
