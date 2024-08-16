# Coworking Space Service Expansion

This project focuses on deploying an analytics API for business analysts using Kubernetes on AWS. The service is part of a microservices-based system for managing coworking spaces, with APIs for token issuance and administrative access control.

## Prerequisites

Ensure you have the following tools and services:

- **Python 3.6+**: For running the application and managing dependencies.
- **Docker CLI**: To build and test Docker images locally.
- **kubectl**: For interacting with the Kubernetes cluster.
- **Helm**: For managing Kubernetes applications.

## Setup and Deployment Process

### 1. Configure PostgreSQL Database

Set up a PostgreSQL database using a Helm chart:

1. **Add Bitnami Helm Repo**:
    ```bash
    helm repo add <REPO_NAME> https://charts.bitnami.com/bitnami
    ```
2. **Install PostgreSQL**:
    ```bash
    helm install <SERVICE_NAME> <REPO_NAME>/postgresql
    ```
3. **Test Database Connection**:
    - **Port Forwarding**:
        ```bash
        kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432 &
        PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
        ```
4. **Run Seed Files**:
    ```bash
    kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < <FILE_NAME.sql>
    ```

### 2. Local Development Setup

1. **Install Python Dependencies**:
    ```bash
    pip install -r requirements.txt
    ```
2. **Run the Application**:
    ```bash
    DB_USERNAME=user DB_PASSWORD=pass python app.py
    ```

### 3. Create the EKS Cluster

Create an EKS cluster to host the application:

```bash
eksctl create cluster --name my-cluster
```

### 4. Configure IAM and CloudWatch Observability

1. **Associate IAM OIDC Provider**:
    ```bash
    eksctl utils associate-iam-oidc-provider --cluster my-cluster --approve
    ```

2. **Create IAM Service Account**:
    ```bash
    eksctl create iamserviceaccount \
    --name cloudwatch-agent \
    --namespace amazon-cloudwatch \
    --cluster my-cluster \
    --role-name EKSCloudWatchRole \
    --attach-policy-arn arn:aws:iam:policy/CloudWatchAgentServerPolicy \
    --role-only \
    --approve
    ```

3. **Install CloudWatch Observability Add-on**:
    ```bash
    aws eks create-addon \
    --addon-name amazon-cloudwatch-observability \
    --cluster-name my-cluster \
    --service-account-role-arn arn:aws:iam:role/EKSCloudWatchRole
    ```

### 5. Deploy the Application to Kubernetes

1. **Apply Kubernetes Deployment**:
    ```bash
    kubectl apply -f deployment.yaml
    ```

2. **Verify the Deployment**:
    - **Check Services**:
        ```bash
        kubectl get svc
        ```
    - **Check Pods**:
        ```bash
        kubectl get pods
        ```
    - **Describe Service**:
        ```bash
        kubectl describe svc <SERVICE_NAME>
        ```
    - **Describe Deployment**:
        ```bash
        kubectl describe deployment <DEPLOYMENT_NAME>
        ```

### Standout Suggestions

1. **Resource Allocation**: Allocate 512Mi memory and 0.5 CPU to optimize performance in Kubernetes.
2. **Instance Type Recommendation**: A `t3.medium` instance balances cost and performance for this deployment.
3. **Cost Optimization**: Use auto-scaling and adjust CloudWatch log retention policies to reduce costs.

### Best Practices

- **Dockerfile**: Use a lightweight base image and clearly comment on complex commands.
- **Versioning**: Follow semantic versioning (e.g., `1.2.1`) for Docker images.
- **Security**: Handle sensitive data using environment variables or AWS Secrets Manager.

## Notes

This README is concise and structured to provide clear guidance on deploying the analytics service, keeping the core instructions within 20 sentences for brevity.