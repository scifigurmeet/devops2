
---

# Amazon Elastic Kubernetes Service (EKS) Tutorial (Updated)

## Introduction

Amazon Elastic Kubernetes Service (EKS) is a **fully managed Kubernetes service** provided by AWS. It allows you to run Kubernetes clusters on AWS **without managing the control plane**. AWS takes care of availability, scalability, and security of the control plane, while you focus on deploying and managing applications.

This guide explains **EKS concepts, cluster creation, deployment, scaling, monitoring, and best practices** using modern AWS recommendations.

---

## Table of Contents

1. Prerequisites
2. Key Concepts
3. Setting Up EKS
4. Deploying an Application
5. Scaling and Updating
6. Monitoring and Logging
7. Best Practices
8. Conclusion

---

## Prerequisites

Before starting with Amazon EKS, ensure you have:

* An AWS account
* AWS CLI v2 installed and configured
* kubectl installed
* eksctl installed
* Basic understanding of Docker and Kubernetes

---

## Key Concepts

* **Cluster**
  A Kubernetes cluster where the control plane is managed by AWS and worker nodes run in your AWS account.

* **Control Plane**
  Includes API server, scheduler, controller manager, and etcd. Managed fully by AWS.

* **Node Group**
  A group of EC2 instances that act as worker nodes for running Pods.

* **Pod**
  The smallest deployable unit in Kubernetes. It runs one or more containers.

* **Service**
  Provides stable networking and access to Pods.

* **Fargate Profile**
  Allows running Pods without managing EC2 instances (serverless compute).

---

## Setting Up EKS

### Step 1: Configure AWS CLI

```bash
aws configure
```

Provide your AWS Access Key, Secret Key, default region, and output format.

---

### Step 2: Create an EKS Cluster using eksctl

```bash
eksctl create cluster --name my-cluster --region us-west-2 --node-type t3.micro --nodes 2
```

This command automatically:

* Creates a VPC and subnets
* Creates IAM roles
* Creates the EKS control plane
* Creates worker nodes
* Configures kubectl

---

### Step 3: Verify Cluster

```bash
kubectl get nodes
```

You should see worker nodes in `Ready` state.

---

## Deploying an Application

### Step 1: Create Deployment YAML

Create a file named `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

---

### Step 2: Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

---

### Step 3: Expose Deployment as Service

```bash
kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80
```

This creates an AWS Load Balancer and exposes your application publicly.

---

### Step 4: Check Service Status

```bash
kubectl get svc
```

Wait until the `EXTERNAL-IP` is assigned.

---

## Scaling and Updating

### Scale Application

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

---

### Update Application Version

1. Update the image version in `deployment.yaml`
2. Apply changes:

```bash
kubectl apply -f deployment.yaml
```

Kubernetes performs a rolling update automatically.

---

## Monitoring and Logging

### Enable Control Plane Logging

```bash
eksctl utils update-cluster-logging --enable-types all --region us-west-2 --cluster my-cluster
```

---

### Monitoring Options

* Amazon CloudWatch Container Insights
* Prometheus and Grafana
* AWS Distro for OpenTelemetry (ADOT)

---

## Best Practices

* Use **IAM Roles for Service Accounts (IRSA)**
* Define CPU and memory requests and limits
* Use **Managed Node Groups**
* Prefer **AWS Load Balancer Controller** for production
* Store secrets in AWS Secrets Manager or SSM Parameter Store
* Regularly upgrade Kubernetes and node AMIs
* Implement CI/CD using GitHub Actions, Jenkins, or AWS CodePipeline

---

## Conclusion

This guide provides a **modern, beginner-friendly introduction to Amazon EKS** with updated tools and best practices.
It is suitable for **students, demos, and first-time EKS users**.

### Clean Up Resources (Important)

```bash
eksctl delete cluster --name my-cluster --region us-west-2
```

This prevents unnecessary AWS charges.

---

Just say the word 👍
