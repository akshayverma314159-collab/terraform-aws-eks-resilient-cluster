# Resilient EKS Architecture (Terraform & Kubernetes)

## Overview
This repository demonstrates a production-ready AWS EKS cluster provisioned via Terraform, focusing on high availability, automated scaling, and cost efficiency. 

Unlike standard tutorials that rely on the AWS console or deploy basic workloads, this project treats infrastructure as code and implements strict Kubernetes reliability controls. This ensures seamless application availability during infrastructure scaling, node failures, and cluster upgrades. The EKS cluster is deployed using Kubernetes version 1.36 to ensure it remains under standard AWS support and avoids extended lifecycle penalties.

## Architecture & SRE Controls
* **Infrastructure as Code (IaC):** A highly available VPC (spanning multiple AZs) and an EKS cluster managed entirely via Terraform.
* **Remote State Management:** Configured to use an S3 backend for state storage with DynamoDB locking, preventing race conditions during collaborative deployments.
* **Access Entry API:** Utilizes the modern `enable_cluster_creator_admin_permissions` flag for secure, IAM-integrated cluster authentication without relying on the legacy `aws-auth` ConfigMap.
* **Horizontal Pod Autoscaling (HPA):** Dynamically scales pod replicas based on CPU utilization to handle traffic spikes automatically.
* **Pod Disruption Budgets (PDB):** Configured to guarantee a minimum number of pods remain available during voluntary disruptions (e.g., node drains or rolling updates).
* **Health Probes:** Strict liveness and readiness probes configured to ensure the load balancer only routes traffic to healthy containers.

## Prerequisites
* AWS CLI installed and configured with appropriate IAM permissions.
* Terraform v1.5+ installed.
* `kubectl` installed and configured.

## Usage Instructions

### 1. Provision the Infrastructure
```bash
# Initialize the Terraform S3 backend and download provider plugins
terraform init

# Review the infrastructure plan
terraform plan

# Provision the AWS resources (VPC, Subnets, EKS Cluster, Node Groups)
terraform apply -auto-approve
```

### 2. Authenticate the Cluster
```bash
# Update your local kubeconfig to interact with the new cluster
aws eks --region ap-south-1 update-kubeconfig --name sre-production-cluster
```

### 3. Deploy the Reliability Controls
```bash
# Apply the deployment, HPA, and PDB manifests
kubectl apply -f kubernetes/

# Verify the scaling and disruption budgets are active
kubectl get hpa
kubectl get pdb
```

### 4. Cleanup
To avoid ongoing AWS charges for the EKS control plane and EC2 nodes, destroy the infrastructure when testing is complete:
```bash
terraform destroy -auto-approve
```
