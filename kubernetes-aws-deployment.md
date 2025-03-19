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

1. Create a GitHub Actions Workflow:

a. Add a .github/workflows/deploy.yml file to your repository.

b. Example deploy.yml:

```
name: Deploy to AWS EKS

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v3
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-west-2

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, tag, and push Docker image to Amazon ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: my-app
          IMAGE_TAG: latest
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

      - name: Install kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubectl for EKS
        run: |
          aws eks update-kubeconfig --name my-cluster --region us-west-2

      - name: Deploy to EKS
        run: |
          kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml

```

2. Add AWS Secrets to GitHub:

i. Go to your GitHub repository > Settings > Secrets > Actions.

ii. Add the following secrets:

iii. AWS_ACCESS_KEY_ID

iv. AWS_SECRET_ACCESS_KEY


**S_tep 3: Run the Pipeline_**

i. Push Code to Main Branch:

a. Push your code to the main branch to trigger the pipeline.

2. Monitor the Pipeline:

i. Go to the Actions tab in your GitHub repository to monitor the pipeline progress.

ii. Example output: 

```
Run docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
Sending build context to Docker daemon  2.048kB
Step 1/5 : FROM node:16
---> 1234567890ab
Step 2/5 : WORKDIR /app
---> Running in 0987654321cd
...
Successfully built 1234567890ab
Successfully tagged <aws-account-id>.dkr.ecr.us-west-2.amazonaws.com/my-app:latest

```

3. Verify Deployment:

i. Check the deployment status in Kubernetes:

```
kubectl get pods
kubectl get svc

```
ii. Example output:

```
NAME                      READY   STATUS    RESTARTS   AGE
my-app-1234567890-abcde   1/1     Running   0          1m
my-app-1234567890-fghij   1/1     Running   0          1m
my-app-1234567890-klmno   1/1     Running   0          1m

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)        AGE
my-app       LoadBalancer   10.100.200.30   a1234567890abcdef1234567890abcdef-1234567890.us-west-2.elb.amazonaws.com   80:3000/TCP   1m

```
**_Step 4: Access the Application_**

Use the EXTERNAL-IP from the kubectl get svc output to access your application.

Example:

```
http://a1234567890abcdef1234567890abcdef-1234567890.us-west-2.elb.amazonaws.com
```
This end-to-end process automates the deployment of the application to AWS EKS using a CI/CD pipeline. 
