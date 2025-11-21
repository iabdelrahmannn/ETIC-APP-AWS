# Microservices Project on AWS EKS

This project demonstrates how to dockerize a Python microservice, provision an AWS EKS cluster using Terraform, deploy the application using Kubernetes manifests, set up a CI/CD pipeline with AWS CodePipeline, and implement monitoring with AWS CloudWatch Container Insights.

## Prerequisites
- AWS CLI configured with appropriate permissions.
- Docker installed.
- Terraform installed.
- `kubectl` installed.

## Step 1: Clone the Repository
```bash
git clone https://github.com/iabdelrahmannn/ETIC-APP-AWS.git
cd ETIC-APP-AWS
```

## Step 2: Dockerize the Application
We created a `dockerfile` to containerize the Flask application.

**Build and Run Locally:**
```bash
docker build -t etic-app-aws:v1 .
docker run -p 5000:5000 etic-app-aws:v1
```

## Step 3: Provision Kubernetes Cluster (EKS)
We used **Terraform** to provision the VPC and EKS cluster.

**Files:**
- `Terraform/main.tf`: Defines VPC and EKS module
- `Terraform/providers.tf`: AWS provider configuration.

**Commands:**
```bash
cd Terraform
terraform init
terraform apply
**Connect to Cluster:**
```bash
aws eks update-kubeconfig --region us-east-1 --name etic-app-cluster
```

## Step 4 & 5: Deploy and Expose Microservice
We defined Kubernetes manifests in the `k8S/` directory.

**Files:**
- `k8S/deployment.yml`: Deploys 2 replicas of the app.
- `k8S/services.yml`: Exposes the app via a LoadBalancer.

**Deploy:**
```bash
kubectl apply -f k8S/
```

**Verify:**
```bash
kubectl get svc etic-app-service
Copy the EXTERNAL-IP and visit it in your browser.
```

## Step 6: CI/CD Pipeline
We implemented a CI/CD pipeline using **AWS CodePipeline** and **AWS CodeBuild**.

**Configuration:**
- **Source**: GitHub Repository.
- **Build**: AWS CodeBuild using `buildspec.yml`.
- **`buildspec.yml`**:
    1.  Installs `kubectl`.
    2.  Logs in to Docker Hub.
    3.  Builds and Pushes Docker image.
    4.  Deploys to EKS using `kubectl apply`.

**Setup Steps:**
1.  Create a CodePipeline in AWS Console.
2.  Connect GitHub repo.
3.  Create CodeBuild project with environment variables:
    - `DOCKERHUB_USERNAME`
    - `DOCKERHUB_PASSWORD`
    - `AWS_REGION`
    - `CLUSTER_NAME`
    - `IMAGE_NAME`

## Step 7: Monitoring (AWS CloudWatch Container Insights)
We configured **Amazon CloudWatch Container Insights** to monitor the cluster metrics and logs.

### Configuration Steps

**1. Attach Policy to Node Role**
The EKS worker nodes need permission to send metrics to CloudWatch.
1.  Go to AWS Console > EC2 > Instances.
2.  Select a worker node -> Security -> Click the IAM Role.
3.  Attach Policy: `CloudWatchAgentServerPolicy`.

**2. Install the CloudWatch Observability Add-on**
The easiest way to enable Container Insights is using the EKS Add-on.

```bash
aws eks create-addon --cluster-name etic-app-cluster --addon-name amazon-cloudwatch-observability
```

**3. Verify**
1.  Go to AWS Console > CloudWatch > Insights > Container Insights.
2.  Select `etic-app-cluster` to view CPU, Memory, and Network metrics.

 7. Cleanup

```bash
cd Terraform
terraform destroy
```
