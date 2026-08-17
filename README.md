# Tech Challenge 2
## Application Deployment: Containerization, Infrastructure as Code, Kubernetes, and CI/CD

## Project Overview

This project demonstrates the deployment of a containerized Python Flask web application to Amazon Web Services using Docker, Amazon ECR, Terraform, Amazon EKS, Kubernetes, Helm, Horizontal Pod Autoscaling, and Jenkins CI/CD.

The application displays:

Hello, World!

The infrastructure is provisioned using Terraform, the application is packaged using Docker, stored in Amazon ECR, and deployed to Amazon EKS.

Jenkins automates the build and deployment workflow by building the Docker image, pushing it to Amazon ECR, connecting to the EKS cluster, and deploying the application using Helm.

---

## Architecture

```text
GitHub
   |
   v
Jenkins
   |
   v
Docker Build
   |
   v
Amazon ECR
   |
   v
Amazon EKS
   |
   v
Helm / Kubernetes
   |
   v
AWS Load Balancer
   |
   v
Flask Application