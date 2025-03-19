# In-house Kubernetes cluster 

This work shows step-by-step guide for deploying an application to an in-house Kubernetes cluster using a CI/CD pipeline. 
It uses Jenkins as the CI/CD tool, Prometheus for monitoring, and Grafana for visualization. 
The process includes setting up Jenkins, deploying Kubernetes, and automating the deployment pipeline.

**- Step 1: Prerequisites**
1. Kubernetes Cluster: Set up an in-house Kubernetes cluster using tools like kubeadm, Minikube, or k3s.

 Example with kubeadm:

```
kubeadm init --pod-network-cidr=10.244.0.0/16
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
Jenkins: Install Jenkins on a server or VM.

2. Jenkins: Install Jenkins on a server or VM.

 ```
sudo apt update
sudo apt install openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

  ```

3. Prometheus and Grafana: Install Prometheus and Grafana for monitoring.

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
helm install grafana grafana/grafana
```


**- Step 2: Prepare the Application**

1. Dockerfile: Ensure your application has a Dockerfile.

Example Dockerfile:

```
FROM node:16
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "app.js"]
```

2. Kubernetes Manifests: Prepare deployment and service YAML files.

Example k8s/deployment.yaml:

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
        image: my-registry/my-app:latest
        ports:
        - containerPort: 3000
```

3. Example k8s/service.yaml:

   ```
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: my-app
   ```

**- Step 3: Set Up Jenkins Pipeline**
1. Install Jenkins Plugins:

i. Install the Kubernetes, Docker Pipeline, and Git plugins via the Jenkins UI.

2. Create a Jenkins Pipeline:

i. Create a new pipeline job in Jenkins.

ii. Add the following Jenkinsfile to your repository:

```
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        script {
          docker.build("my-registry/my-app:${env.BUILD_ID}")
        }
      }
    }
    stage('Push to Registry') {
      steps {
        script {
          docker.withRegistry('https://my-registry', 'registry-credentials') {
            docker.image("my-registry/my-app:${env.BUILD_ID}").push()
          }
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        sh 'kubectl apply -f k8s/deployment.yaml'
        sh 'kubectl apply -f k8s/service.yaml'
      }
    }
  }
}
```

3. Configure Registry Credentials:

i. Add your Docker registry credentials in Jenkins under Credentials > System > Global Credentials.
