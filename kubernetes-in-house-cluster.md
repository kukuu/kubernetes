# In-house Kubernetes Cluster 

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


**- Step 4: Run the Pipeline**

1. Trigger the Pipeline:

i. Push your code to the repository to trigger the Jenkins pipeline.

```
git add .
git commit -m "Trigger CI/CD pipeline"
git push origin main

```

2. Monitor the Pipeline:

i. Check the Jenkins console output for progress.

ii. Example output:

```
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] script
[Pipeline] {
[Pipeline] docker
Successfully built 1234567890ab
Successfully tagged my-registry/my-app:1
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Push to Registry)
[Pipeline] script
[Pipeline] {
[Pipeline] docker
The push refers to repository [my-registry/my-app]
...

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
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
my-app       NodePort    10.100.200.30   <none>        80:30000/TCP   1m
```
**- Step 5: Set Up Monitoring** 

i. Access Prometheus:

ii. Port-forward the Prometheus service:

```
kubectl port-forward svc/prometheus-server 9090:9090
```
iii. Open http://localhost:9090 to view metrics.

2. Access Grafana:

i. Port-forward the Grafana service:

```

kubectl port-forward svc/grafana 3000:3000

```
ii. Open http://localhost:3000 and log in with the default credentials (admin/admin).

iii. Add Prometheus as a data source and create dashboards to visualize metrics.

-** Step 6: Access the Application**

i.  Use the NodePort from the kubectl get svc output to access your application.

ii.  Example:

```
http://<node-ip>:30000

```
