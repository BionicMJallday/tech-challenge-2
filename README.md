# Tech Challenge 2 — Deploying Microservices on Amazon EKS

## Project Overview

This project demonstrates the deployment of a containerized Python Flask application to Amazon Elastic Kubernetes Service (EKS) using Infrastructure as Code, Kubernetes, Helm, and a Jenkins CI/CD pipeline.

The application returns:

```text
Hello, World!
```

The project uses Terraform to provision the AWS infrastructure, Amazon ECR to store the Docker image, Amazon EKS to run the Kubernetes workload, Kubernetes Horizontal Pod Autoscaling for application scaling, Helm for application packaging and deployment, Jenkins for CI/CD automation, and an AWS Application Load Balancer (ALB) for external application access.

A separate `gitops` branch also contains an alternative GitOps configuration using GitHub Actions and Argo CD.

---

## Architecture

```text
Developer
   |
   v
GitHub Repository
   |
   v
Jenkins CI/CD
   |
   +----------------------+
   |                      |
   v                      v
Docker Build         Helm Deployment
   |                      |
   v                      |
Amazon ECR                |
                          v
                     Amazon EKS
                          |
                          v
                Kubernetes Deployment
                          |
                          v
                  Kubernetes Service
                          |
                          v
                 Kubernetes Ingress
                          |
                          v
          AWS Application Load Balancer
                          |
                          v
                   Flask Application
                          |
                          v
                    Hello, World!
```

---

## Technologies Used

- AWS
- Amazon EKS
- Amazon ECR
- Amazon EC2
- AWS Application Load Balancer
- AWS Load Balancer Controller
- Terraform
- Docker
- Kubernetes
- Helm
- Jenkins
- Git
- GitHub
- Python
- Flask
- GitHub Actions
- Argo CD

---

## Project Structure

```text
tech-challenge-2/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── README.md
│
├── terraform/
│   ├── main.tf
│   ├── providers.tf
│   └── versions.tf
│
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── hpa.yaml
│   └── ingress.yaml
│
└── helm/
    └── tc2-app/
        ├── Chart.yaml
        ├── values.yaml
        └── templates/
```

The separate `gitops` branch additionally contains:

```text
.github/
└── workflows/
    └── gitops.yml

argocd/
└── application.yaml
```

---

# Application

The project uses a lightweight Python Flask application.

The application listens on port `5000` and returns:

```text
Hello, World!
```

The Python dependencies are defined in `requirements.txt`.

---

# Docker

The Flask application is packaged as a Docker container.

The Docker image is built from the project `Dockerfile` and stored in Amazon Elastic Container Registry (ECR).

ECR repository:

```text
tech-challenge-2-app
```

Image URI:

```text
377730029945.dkr.ecr.us-east-1.amazonaws.com/tech-challenge-2-app:latest
```

---

# Infrastructure as Code with Terraform

Terraform is used to provision the AWS infrastructure required by the application.

The Terraform configuration creates and configures the networking and Amazon EKS environment required to run the Kubernetes workload.

The infrastructure includes:

- Dedicated VPC
- Public and private subnets
- Multiple Availability Zones
- NAT Gateway
- DNS support
- Amazon EKS cluster
- EKS managed node group
- Kubernetes subnet tagging
- EKS system add-ons

The required EKS system components include:

- Amazon VPC CNI
- CoreDNS
- kube-proxy

---

## EKS Managed Node Group

The EKS managed node group uses:

```text
Instance Type: t3.small

Minimum Nodes: 1
Desired Nodes: 1
Maximum Nodes: 4
```

The final node group was verified as:

```text
Status: ACTIVE
Health: No Issues
```

The Kubernetes worker node was also verified as:

```text
Ready
```

This provides one active worker node while defining node-group capacity up to four `t3.small` instances.

---

# Kubernetes Deployment

The application is deployed to Amazon EKS using Kubernetes manifests stored in the `kubernetes` directory.

The Kubernetes Deployment:

- Uses the Docker image stored in Amazon ECR
- Runs one initial application replica
- Exposes container port `5000`
- Defines CPU requests and limits
- Defines memory requests and limits

The application pod was successfully verified as:

```text
READY:    1/1
STATUS:   Running
RESTARTS: 0
```

---

# Kubernetes Service

A Kubernetes Service exposes the Flask application.

The service routes:

```text
Port 80
   |
   v
Target Port 5000
   |
   v
Flask Application
```

The service is named:

```text
tc2-app-service
```

---

# Horizontal Pod Autoscaling

A Kubernetes Horizontal Pod Autoscaler (HPA) is configured for the application.

Scaling configuration:

```text
Minimum Replicas: 1
Maximum Replicas: 3

CPU Target:    50%
Memory Target: 50%
```

The Metrics Server provides resource utilization information to Kubernetes.

The HPA was successfully verified with live CPU and memory metrics.

Example verified state:

```text
NAME          REFERENCE            TARGETS
tc2-app-hpa   Deployment/tc2-app   cpu: 1%/50%, memory: 16%/50%
```

This allows Kubernetes to adjust application replica count based on resource utilization.

---

# Helm Deployment

The application is also packaged as a Helm chart under:

```text
helm/tc2-app
```

The chart was validated using:

```bash
helm lint ./tc2-app
```

The chart passed validation successfully.

The application was deployed using:

```bash
helm upgrade --install tc2-helm ./tc2-app
```

The Helm release was successfully verified as:

```text
NAME:      tc2-helm
STATUS:    deployed
CHART:     tc2-app-0.1.0
```

---

# Jenkins CI/CD Pipeline

Jenkins provides the primary CI/CD implementation for the project.

The Jenkins pipeline is defined in:

```text
Jenkinsfile
```

The Jenkins server is configured with the tools required to interact with AWS and the EKS environment, including:

- Docker
- AWS CLI
- kubectl
- Helm

The Jenkins EC2 instance uses an AWS IAM role to interact with the required AWS services.

The pipeline performs the automated workflow required to build and deploy the application.

```text
GitHub
   |
   v
Jenkins
   |
   v
Build Docker Image
   |
   v
Authenticate to Amazon ECR
   |
   v
Push Docker Image
   |
   v
Connect to Amazon EKS
   |
   v
Deploy with Helm
```

The Jenkins server was successfully verified as being able to access the EKS cluster:

```text
kubectl get nodes
```

returned the EKS worker node in:

```text
Ready
```

The Jenkins pipeline successfully completed the application deployment.

---

# AWS Application Load Balancer

The AWS Load Balancer Controller was installed in the EKS cluster using Helm.

The controller was verified with both controller pods in:

```text
Running
```

A Kubernetes Ingress resource named:

```text
tc2-app-ingress
```

uses the `alb` Ingress class.

The Ingress configuration provisions an internet-facing AWS Application Load Balancer using IP targets.

The final ALB endpoint is:

```text
k8s-default-tc2appin-4bc5021eb5-806517385.us-east-1.elb.amazonaws.com
```

AWS verified the load balancer as:

```text
Type:   application
Scheme: internet-facing
State:  active
```

The AWS Load Balancer Controller also created a TargetGroupBinding connecting:

```text
AWS ALB
   |
   v
tc2-app-service:80
   |
   v
Flask Pod:5000
```

---

# Deployed Application

The application was externally tested through the AWS Application Load Balancer using:

```bash
curl http://k8s-default-tc2appin-4bc5021eb5-806517385.us-east-1.elb.amazonaws.com
```

Successful response:

```text
Hello, World!
```

This verified the complete traffic path:

```text
Internet
   |
   v
AWS Application Load Balancer
   |
   v
Kubernetes Ingress
   |
   v
Kubernetes Service
   |
   v
Kubernetes Deployment
   |
   v
Flask Application Pod
   |
   v
Hello, World!
```

---

# GitOps Alternative

The primary verified CI/CD implementation is maintained on the `main` branch using Jenkins.

A separate branch named:

```text
gitops
```

contains an alternative GitOps configuration.

The branch includes:

```text
.github/workflows/gitops.yml
argocd/application.yaml
```

The GitHub Actions workflow defines a CI process designed to:

```text
Git Push
   |
   v
GitHub Actions
   |
   v
Build Docker Image
   |
   v
Authenticate to AWS
   |
   v
Push Image to Amazon ECR
   |
   v
Update Helm Image Tag
```

The Argo CD Application manifest is configured to use:

```text
Branch: gitops
Path:   helm/tc2-app
```

The intended GitOps deployment flow is:

```text
GitHub gitops Branch
   |
   v
Argo CD
   |
   v
Helm Chart
   |
   v
Amazon EKS
```

The GitOps configuration is maintained separately so that it does not interfere with the primary Jenkins implementation.

---

# Troubleshooting

Several infrastructure issues were identified and resolved during the project.

## EKS NodeCreationFailure

An initial EKS managed node group failed because the Kubernetes worker did not become healthy.

The Terraform configuration was updated to explicitly manage the required EKS system add-ons, including deploying the Amazon VPC CNI before compute resources.

After correcting the cluster configuration, the replacement worker successfully joined Kubernetes.

## EC2 vCPU Quota

A subsequent node-group creation attempt failed with:

```text
VcpuLimitExceeded
```

The AWS account had an 8-vCPU EC2 quota, and failed EKS node groups were still consuming EC2 capacity.

The failed node groups were identified and deleted. After releasing the unused capacity and reconciling Terraform with AWS, the managed node group deployed successfully.

## Metrics API

The Kubernetes HPA initially reported:

```text
<unknown>
```

for CPU and memory utilization because the Metrics API was not yet available.

After configuring the Metrics Server, the HPA successfully reported live CPU and memory utilization.

## ALB DNS Provisioning

Immediately after creating the Kubernetes Ingress, the ALB DNS hostname was temporarily unavailable while AWS completed load balancer provisioning.

AWS CLI verification showed the Application Load Balancer transitioning through the provisioning process.

After provisioning completed, the ALB DNS endpoint successfully returned:

```text
Hello, World!
```

---

# Final Verification

The completed environment was verified using:

```bash
kubectl get nodes
kubectl get deployments
kubectl get pods
kubectl get services
kubectl get hpa
kubectl get ingress
helm list
```

The final environment confirmed:

- EKS worker node `Ready`
- EKS managed node group `ACTIVE`
- Application Deployment available
- Application pods `Running`
- Kubernetes Service available
- HPA reporting CPU and memory utilization
- Helm release `deployed`
- AWS Application Load Balancer provisioned
- External application returning `Hello, World!`
- Jenkins CI/CD pipeline successfully deploying the application

---

# Branch Strategy

## `main`

Contains the primary, verified Jenkins CI/CD implementation.

## `gitops`

Contains the alternative GitHub Actions and Argo CD GitOps configuration.

---

# Project Result

**SUCCESS — VERIFIED**

Tech Challenge 2 successfully demonstrates a containerized Flask application deployed to Amazon EKS using a cloud-native DevOps workflow.

The project includes:

- Docker containerization
- Amazon ECR image storage
- Terraform Infrastructure as Code
- Amazon EKS
- EKS managed node-group capacity from 1 to 4 `t3.small` nodes
- Kubernetes Deployments and Services
- Horizontal Pod Autoscaling
- Helm application packaging
- Jenkins CI/CD
- AWS Application Load Balancer
- Kubernetes Ingress
- Separate GitOps branch configuration

The final Flask application was successfully accessed through the AWS Application Load Balancer and returned:

```text
Hello, World!
```