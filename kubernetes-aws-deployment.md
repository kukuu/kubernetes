# Kubernetes on AWS using a CI/CD pipeline.

This **BLUEPRINT** work exemplifies a step-by-step guide for deploying an application to Kubernetes on AWS using a CI/CD pipeline. It uses GitHub Actions as the CI/CD tool, AWS EKS (Elastic Kubernetes Service) for Kubernetes, and Docker for containerization. The process includes building the Docker image, pushing it to AWS ECR (Elastic Container Registry), and deploying it to EKS.

**_Step 1: Prerequisites_**

1. AWS Account: Ensure you have an AWS account with permissions for EKS, ECR, and IAM.

2. EKS Cluster: Create an EKS cluster using the AWS CLI or Console.

```
eksctl create cluster --name my-cluster --region us-west-2 --nodegroup-name my-nodes --node-type t3.medium --nodes 3
```

3. Dockerfile: Ensure your application has a Dockerfile.
   
Example Dockerfile:

```
FROM node:16
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "app.js"]
```

4. Kubernetes Manifests: Prepare deployment and service YAML files.

i. Example k8s/deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: my-app
        image: <aws-account-id>.dkr.ecr.<region>.amazonaws.com/my-app:latest
        ports:
        - containerPort: 3000

```

ii. Example k8s/service.yaml: 

```
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: my-app
```

**_Step 2: Set Up GitHub Actions CI/CD Pipeline_**
