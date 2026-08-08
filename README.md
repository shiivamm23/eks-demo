# 🚀 Frontend CI/CD Deployment on Amazon EKS Using Jenkins, Amazon ECR & AWS Load Balancer Controller

![AWS](https://img.shields.io/badge/AWS-Cloud-orange)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red)
![Docker](https://img.shields.io/badge/Docker-Containerization-blue)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-326CE5)
![Amazon EKS](https://img.shields.io/badge/Amazon-EKS-FF9900)
![Amazon ECR](https://img.shields.io/badge/Amazon-ECR-FF9900)
![Helm](https://img.shields.io/badge/Helm-Package%20Manager-0F1689)

## 📖 Project Overview

This project demonstrates an **end-to-end CI/CD pipeline** for deploying a containerized frontend application to **Amazon Elastic Kubernetes Service (EKS)** using **Jenkins, Docker, Amazon Elastic Container Registry (ECR), Kubernetes, Helm, and AWS Load Balancer Controller**.

The pipeline automates the complete application delivery process:

**Developer → GitHub → Jenkins → Docker → Amazon ECR → Amazon EKS → AWS Load Balancer Controller → Application Load Balancer → End Users**

Instead of using Docker Hub, container images are securely stored in **Amazon ECR**. The application is deployed to an **Amazon EKS** cluster, while the **AWS Load Balancer Controller** automatically provisions an **Application Load Balancer (ALB)** based on Kubernetes Ingress resources.

This project demonstrates practical experience with:

* CI/CD automation
* Docker containerization
* Jenkins pipelines
* Amazon ECR
* Amazon EKS
* Kubernetes Deployments and Services
* Kubernetes Ingress
* Helm
* AWS Load Balancer Controller
* AWS IAM and access management
* Cloud-based application deployment

---

# 🏗️ Architecture

```text
                         Developer
                             │
                             │ Git Push
                             ▼
                        GitHub Repository
                             │
                             │ Webhook
                             ▼
                    Jenkins Server (EC2)
                             │
                             ▼
                    Checkout Source Code
                             │
                             ▼
                     Build Frontend App
                             │
                             ▼
                      Build Docker Image
                             │
                             ▼
                    Authenticate with ECR
                             │
                             ▼
                  Push Image to Amazon ECR
                             │
                             ▼
                    Configure kubectl
                             │
                             ▼
                 Deploy Manifests to EKS
                             │
                             ▼
                    Amazon EKS Cluster
                             │
                 ┌───────────┼───────────┐
                 │           │           │
                 ▼           ▼           ▼
             Deployment   Service     Ingress
                                         │
                                         ▼
                           AWS Load Balancer Controller
                                         │
                                         ▼
                           Application Load Balancer
                                         │
                                         ▼
                                    End Users
```

---

# 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [Architecture](#-architecture)
3. [Technologies Used](#-technologies-used)
4. [Prerequisites](#-prerequisites)
5. [AWS Infrastructure Setup](#-aws-infrastructure-setup)
6. [Jenkins Server Setup](#-jenkins-server-setup)
7. [Amazon EKS Cluster Setup](#-amazon-eks-cluster-setup)
8. [Amazon ECR Setup](#-amazon-ecr-setup)
9. [Helm Installation](#-helm-installation)
10. [AWS Load Balancer Controller](#-aws-load-balancer-controller)
11. [Jenkins Configuration](#-jenkins-configuration)
12. [Kubernetes Manifests](#-kubernetes-manifests)
13. [CI/CD Pipeline](#-cicd-pipeline)
14. [Deployment Verification](#-deployment-verification)
15. [Troubleshooting](#-troubleshooting)
16. [Cleanup](#-cleanup)
17. [Learning Outcomes](#-learning-outcomes)
18. [Conclusion](#-conclusion)

---

# 🛠️ Technologies Used

| Technology                       | Purpose                      |
| -------------------------------- | ---------------------------- |
| **GitHub**                       | Source Code Management       |
| **Jenkins**                      | CI/CD Automation             |
| **Docker**                       | Application Containerization |
| **Amazon EC2**                   | Jenkins Server               |
| **Amazon ECR**                   | Container Image Registry     |
| **Amazon EKS**                   | Managed Kubernetes Cluster   |
| **Kubernetes**                   | Container Orchestration      |
| **kubectl**                      | Kubernetes Command-Line Tool |
| **eksctl**                       | EKS Cluster Management       |
| **Helm**                         | Kubernetes Package Manager   |
| **AWS Load Balancer Controller** | AWS ALB Provisioning         |
| **Application Load Balancer**    | External Application Access  |
| **Node.js & npm**                | Frontend Build               |
| **AWS CLI**                      | AWS Resource Management      |

---

# 📌 Prerequisites

Before starting this project, ensure you have:

* An active AWS account
* A GitHub account
* An IAM user or IAM role with appropriate permissions
* An Ubuntu EC2 instance
* Docker installed
* AWS CLI installed and configured
* `kubectl` installed
* `eksctl` installed
* Helm installed
* Basic knowledge of:

  * Linux
  * Git and GitHub
  * Docker
  * Kubernetes
  * AWS

---

# ☁️ AWS Infrastructure Setup

## Step 1 — Create IAM User / Role

Create an IAM identity with the required permissions.

For a lab environment, the following managed policies may be used where appropriate:

* `AmazonEKSClusterPolicy`
* `AmazonEKSWorkerNodePolicy`
* `AmazonEC2ContainerRegistryFullAccess`
* `IAMFullAccess`

> **Security Note:** For production environments, avoid broad permissions such as `IAMFullAccess`. Follow the principle of least privilege and create dedicated IAM roles and policies.

---

## Step 2 — Launch Jenkins EC2 Instance

Recommended configuration:

| Property      | Recommended Value |
| ------------- | ----------------- |
| AMI           | Ubuntu 22.04 LTS  |
| Instance Type | `t3.medium`       |
| Storage       | 30 GB or more     |
| SSH Port      | `22`              |
| HTTP Port     | `80`              |
| HTTPS Port    | `443`             |
| Jenkins Port  | `8080`            |

Connect to the instance:

```bash
ssh -i key.pem ubuntu@<EC2-PUBLIC-IP>
```

---

# ⚙️ Jenkins Server Setup

## Step 1 — Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

---

## Step 2 — Install Java

Jenkins requires Java.

```bash
sudo apt install openjdk-17-jdk -y
```

Verify:

```bash
java -version
```

---

## Step 3 — Install Jenkins

Add the Jenkins repository:

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Update the package repository:

```bash
sudo apt update
```

Install Jenkins:

```bash
sudo apt install jenkins -y
```

Enable and start Jenkins:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Check Jenkins status:

```bash
sudo systemctl status jenkins
```

Jenkins should now be accessible at:

```text
http://<EC2-PUBLIC-IP>:8080
```

Retrieve the initial administrator password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# 🐳 Install Docker

Install Docker:

```bash
sudo apt install docker.io -y
```

Enable and start Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify:

```bash
docker --version
```

---

## Allow Jenkins to Use Docker

Add the Jenkins user to the Docker group:

```bash
sudo usermod -aG docker jenkins
```

Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

Verify Docker access:

```bash
sudo -u jenkins docker ps
```

If required, log out and reconnect to the EC2 instance for group membership changes to take effect.

---

# 🟢 Install Node.js and npm

```bash
sudo apt install nodejs npm -y
```

Verify:

```bash
node -v
npm -v
```

---

# ☁️ Install and Configure AWS CLI

Verify whether AWS CLI is installed:

```bash
aws --version
```

Configure AWS credentials:

```bash
aws configure
```

Provide:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name: ap-south-1
Default output format: json
```

Verify authentication:

```bash
aws sts get-caller-identity
```

> **Security Recommendation:** For EC2-based Jenkins deployments, prefer attaching an IAM role to the EC2 instance instead of storing long-lived AWS access keys.

---

# ☸️ Install kubectl

Download the latest stable kubectl release:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Install:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

---

# 🚀 Install eksctl

Download and install `eksctl`:

```bash
curl --silent --location \
"https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
| tar xz -C /tmp
```

```bash
sudo mv /tmp/eksctl /usr/local/bin
```

Verify:

```bash
eksctl version
```

---

# ☸️ Amazon EKS Cluster Setup

## Step 1 — Create the EKS Cluster

Example:

```bash
eksctl create cluster \
  --name demo-cluster \
  --region ap-south-1 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2
```

This creates:

* EKS control plane
* Managed worker nodes
* VPC networking
* Subnets
* Security groups
* IAM roles
* Kubernetes configuration

---

## Step 2 — Verify the Cluster

Check cluster information:

```bash
kubectl cluster-info
```

Check worker nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                         STATUS   ROLES    AGE
ip-xxx-xxx-xxx-xxx           Ready    <none>   ...
ip-xxx-xxx-xxx-xxx           Ready    <none>   ...
```

---

# 🔐 Configure EKS Access

If the Jenkins EC2 instance uses an IAM role that needs access to the EKS cluster, create an EKS access entry.

Example:

```bash
aws eks create-access-entry \
  --cluster-name demo-cluster \
  --principal-arn arn:aws:iam::<ACCOUNT_ID>:role/<EC2-IAM-ROLE> \
  --type STANDARD \
  --region ap-south-1
```

Associate an appropriate EKS access policy:

```bash
aws eks associate-access-policy \
  --cluster-name demo-cluster \
  --principal-arn arn:aws:iam::<ACCOUNT_ID>:role/<EC2-IAM-ROLE> \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy \
  --access-scope type=cluster \
  --region ap-south-1
```

> **Production Recommendation:** Use a more restrictive EKS access policy rather than `AmazonEKSClusterAdminPolicy` whenever possible.

---

# 📦 Amazon ECR Setup

Amazon ECR is used to store the Docker image generated by Jenkins.

## Step 1 — Create ECR Repository

```bash
aws ecr create-repository \
  --repository-name frontend \
  --region ap-south-1
```

Verify:

```bash
aws ecr describe-repositories \
  --repository-names frontend \
  --region ap-south-1
```

---

## Step 2 — Authenticate Docker with ECR

```bash
aws ecr get-login-password \
  --region ap-south-1 | \
  docker login \
  --username AWS \
  --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
```

---

## Step 3 — Build Docker Image

From the frontend project directory:

```bash
docker build -t frontend .
```

Verify:

```bash
docker images
```

---

## Step 4 — Tag the Docker Image

```bash
docker tag frontend:latest \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/frontend:latest
```

---

## Step 5 — Push Image to ECR

```bash
docker push \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/frontend:latest
```

Verify:

```bash
aws ecr list-images \
  --repository-name frontend \
  --region ap-south-1
```

---

# ⛵ Install Helm

Helm is used to install and manage Kubernetes applications and components.

Install Helm using the official installation method for your environment.

Verify:

```bash
helm version
```

Add the AWS EKS Helm repository:

```bash
helm repo add eks https://aws.github.io/eks-charts
```

Update repositories:

```bash
helm repo update
```

---

# 🌐 AWS Load Balancer Controller

The **AWS Load Balancer Controller** allows Kubernetes Ingress resources to automatically provision AWS Application Load Balancers.

---

## Step 1 — Associate OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster \
  --region ap-south-1 \
  --approve
```

---

## Step 2 — Download IAM Policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

---

## Step 3 — Create IAM Policy

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

If the policy already exists, retrieve its ARN instead of creating it again.

---

## Step 4 — Create IAM Service Account

Example:

```bash
eksctl create iamserviceaccount \
  --cluster demo-cluster \
  --region ap-south-1 \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve
```

---

## Step 5 — Install AWS Load Balancer Controller

```bash
helm install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1
```

---

## Step 6 — Verify Controller

Check the deployment:

```bash
kubectl get deployment \
  aws-load-balancer-controller \
  -n kube-system
```

Check the pods:

```bash
kubectl get pods -n kube-system
```

Expected:

```text
aws-load-balancer-controller-xxxxx   Running
```

---

# ⚙️ Jenkins Configuration

## Required Jenkins Plugins

Install the following plugins:

* Git
* Pipeline
* Docker Pipeline
* Credentials Binding
* NodeJS
* Kubernetes CLI
* GitHub Integration

Depending on the Jenkins setup, additional AWS-related plugins may be used.

---

# 🔐 Jenkins Credentials

Configure the required credentials under:

```text
Jenkins
→ Manage Jenkins
→ Credentials
```

Typical credentials include:

* GitHub credentials/token
* Docker credentials if Docker Hub is used
* AWS credentials if static credentials are being used

> **Recommended:** When Jenkins runs on AWS EC2, use an IAM role attached to the EC2 instance whenever possible rather than storing AWS access keys in Jenkins.

---

# 🔄 Create Jenkins Pipeline

Create a new Jenkins Pipeline job.

Configure:

```text
Pipeline
→ Definition
→ Pipeline script from SCM
```

SCM:

```text
Git
```

Repository:

```text
https://github.com/<USERNAME>/<REPOSITORY>.git
```

Jenkinsfile:

```text
Jenkinsfile
```

Enable GitHub webhook integration if automatic builds are required.

---

# 📂 Kubernetes Project Structure

Recommended repository structure:

```text
frontend-project/
│
├── src/
│   ├── components/
│   └── ...
│
├── public/
│
├── Dockerfile
├── Jenkinsfile
├── package.json
├── package-lock.json
│
├── deployment.yaml
├── service.yaml
├── ingress.yaml
│
└── README.md
```

---

# ☸️ Kubernetes Manifests

The application uses three primary Kubernetes resources:

### Deployment

Responsible for running and managing the frontend application pods.

```text
deployment.yaml
```

### Service

Provides internal network access to the frontend pods.

```text
service.yaml
```

### Ingress

Defines external HTTP/HTTPS routing and allows the AWS Load Balancer Controller to provision an ALB.

```text
ingress.yaml
```

---

# 🔄 CI/CD Pipeline Workflow

The Jenkins pipeline follows this workflow:

```text
Developer
    │
    ▼
Git Push
    │
    ▼
GitHub Repository
    │
    ▼
GitHub Webhook
    │
    ▼
Jenkins
    │
    ▼
Checkout Source Code
    │
    ▼
Install Dependencies / Build Frontend
    │
    ▼
Build Docker Image
    │
    ▼
Authenticate with Amazon ECR
    │
    ▼
Tag Docker Image
    │
    ▼
Push Image to Amazon ECR
    │
    ▼
Configure kubectl
    │
    ▼
Deploy to Amazon EKS
    │
    ▼
Kubernetes Deployment
    │
    ▼
Kubernetes Service
    │
    ▼
Kubernetes Ingress
    │
    ▼
AWS Load Balancer Controller
    │
    ▼
Application Load Balancer
    │
    ▼
Frontend Application
```

---

# 🧩 CI/CD Pipeline Stages

A typical Jenkinsfile can contain the following stages:

```text
1. Checkout
2. Install Dependencies
3. Build Application
4. Docker Build
5. ECR Authentication
6. Docker Image Tagging
7. Push Image to ECR
8. Configure Kubernetes Access
9. Deploy to EKS
10. Verify Rollout
```

---

# 🔍 Deployment Verification

After the Jenkins pipeline completes successfully, verify the Kubernetes resources.

## Check Pods

```bash
kubectl get pods
```

---

## Check Deployment

```bash
kubectl get deployment
```

---

## Check Services

```bash
kubectl get svc
```

---

## Check Ingress

```bash
kubectl get ingress
```

The Ingress should eventually display an AWS Load Balancer address.

Example:

```text
NAME              CLASS    HOSTS   ADDRESS
frontend-ingress  alb      *       k8s-frontend-xxxxx.ap-south-1.elb.amazonaws.com
```

---

## Check Kubernetes Events

```bash
kubectl get events
```

For more detailed information:

```bash
kubectl describe ingress <INGRESS-NAME>
```

---

## Check Application Logs

```bash
kubectl logs <POD-NAME>
```

For continuous logs:

```bash
kubectl logs -f <POD-NAME>
```

---

# 🧪 Useful Kubernetes Commands

List all resources:

```bash
kubectl get all
```

List pods with node information:

```bash
kubectl get pods -o wide
```

Describe a pod:

```bash
kubectl describe pod <POD-NAME>
```

Describe deployment:

```bash
kubectl describe deployment <DEPLOYMENT-NAME>
```

Restart a deployment:

```bash
kubectl rollout restart deployment <DEPLOYMENT-NAME>
```

Check rollout status:

```bash
kubectl rollout status deployment <DEPLOYMENT-NAME>
```

View deployment history:

```bash
kubectl rollout history deployment <DEPLOYMENT-NAME>
```

---

# ❗ Troubleshooting

## 1. Docker Permission Denied

If Jenkins cannot access Docker:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

Verify:

```bash
sudo -u jenkins docker ps
```

---

## 2. kubectl Cannot Connect to EKS

Update the kubeconfig:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name demo-cluster
```

Verify:

```bash
kubectl get nodes
```

---

## 3. ImagePullBackOff

If the pod shows:

```text
ImagePullBackOff
```

Check:

* ECR repository name
* Docker image URI
* Docker image tag
* EKS node IAM permissions
* ECR repository permissions
* Kubernetes deployment image configuration

Check pod details:

```bash
kubectl describe pod <POD-NAME>
```

---

## 4. CrashLoopBackOff

Check pod logs:

```bash
kubectl logs <POD-NAME>
```

Then inspect the pod:

```bash
kubectl describe pod <POD-NAME>
```

Common causes include:

* Application startup failure
* Incorrect environment variables
* Incorrect container port
* Missing dependencies
* Application configuration errors

---

## 5. Ingress Address Stuck on `<pending>`

Check the AWS Load Balancer Controller:

```bash
kubectl get pods -n kube-system
```

Check controller logs:

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller
```

Verify:

* AWS Load Balancer Controller is running
* IAM role is correctly configured
* OIDC provider is associated
* IAM policy is attached
* EKS subnets are correctly tagged
* Ingress configuration is valid

Check Ingress events:

```bash
kubectl describe ingress <INGRESS-NAME>
```

---

## 6. Jenkins Pipeline Fails During ECR Login

Verify AWS authentication:

```bash
aws sts get-caller-identity
```

Verify ECR access:

```bash
aws ecr describe-repositories \
  --repository-names frontend \
  --region ap-south-1
```

Test Docker authentication:

```bash
aws ecr get-login-password \
  --region ap-south-1 | \
  docker login \
  --username AWS \
  --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
```

---

# 🧹 Cleanup

To avoid unnecessary AWS charges, delete the resources after completing the project.

## Delete Kubernetes Resources

```bash
kubectl delete -f ingress.yaml
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

---

## Delete EKS Cluster

```bash
eksctl delete cluster \
  --name demo-cluster \
  --region ap-south-1
```

---

## Delete ECR Repository

```bash
aws ecr delete-repository \
  --repository-name frontend \
  --region ap-south-1 \
  --force
```

---

## Terminate Jenkins EC2 Instance

Terminate the Jenkins EC2 instance from the AWS EC2 console after the project is complete.

---

# 🎯 Learning Outcomes

By completing this project, you gain practical experience with:

### CI/CD

* Jenkins Pipeline
* GitHub Webhooks
* Automated application builds
* Continuous Integration
* Continuous Deployment

### Docker

* Dockerfile creation
* Docker image building
* Image tagging
* Container image management
* Amazon ECR integration

### Kubernetes

* Kubernetes Deployments
* Kubernetes Pods
* Kubernetes Services
* Kubernetes Ingress
* Rolling deployments
* Application troubleshooting
* Kubernetes networking

### Amazon EKS

* EKS cluster creation
* Worker node management
* EKS access configuration
* `kubectl` administration
* Kubernetes workload deployment

### AWS

* Amazon EC2
* Amazon ECR
* Amazon EKS
* IAM
* Application Load Balancer
* AWS Load Balancer Controller

### Helm

* Helm repositories
* Helm charts
* Kubernetes component installation
* Controller management

---

# 📊 Project Highlights

This project demonstrates an end-to-end DevOps deployment workflow:

```text
GitHub
   ↓
Jenkins
   ↓
Docker
   ↓
Amazon ECR
   ↓
Amazon EKS
   ↓
Kubernetes
   ↓
AWS Load Balancer Controller
   ↓
Application Load Balancer
   ↓
End Users
```

### Key DevOps Practices Demonstrated

* Source control with Git/GitHub
* Automated CI/CD with Jenkins
* Containerization with Docker
* Container registry using Amazon ECR
* Kubernetes orchestration using Amazon EKS
* Infrastructure and access management using AWS IAM
* Kubernetes package management using Helm
* Ingress-based application exposure
* Automated ALB provisioning
* Deployment verification and troubleshooting

---

# 🎉 Conclusion

This project demonstrates a **production-oriented CI/CD workflow** for deploying a containerized frontend application to **Amazon EKS using Jenkins, Docker, Amazon ECR, Kubernetes, Helm, and AWS Load Balancer Controller**.

The pipeline automates the application delivery lifecycle from source-code changes in GitHub to a running application on Amazon EKS.

Docker provides application containerization, Amazon ECR securely stores container images, Jenkins automates the CI/CD workflow, Amazon EKS provides managed Kubernetes orchestration, and the AWS Load Balancer Controller integrates Kubernetes Ingress with AWS Application Load Balancers.

The resulting architecture provides a scalable and automated deployment workflow and demonstrates practical knowledge of modern **DevOps, Kubernetes, AWS, and CI/CD practices**.

---

# 👨‍💻 Author

**SHIVAM UPADHYAY**

### Project

**Frontend CI/CD Deployment on Amazon EKS using Jenkins, Amazon ECR & AWS Load Balancer Controller**

### Technologies

`AWS` `EKS` `ECR` `EC2` `IAM` `Jenkins` `Docker` `Kubernetes` `Helm` `GitHub` `Node.js`

---

