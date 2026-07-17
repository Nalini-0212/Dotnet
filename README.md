# 🚀 .NET DevSecOps Project using Jenkins, SonarQube, Trivy, Docker & AWS EKS

## 📌 Project Description

This project demonstrates the implementation of a complete DevSecOps CI/CD pipeline for a .NET application using industry-standard tools and AWS cloud services.

The pipeline automates:

- Source Code Management
- Continuous Integration
- Code Quality Analysis
- Security Scanning
- Docker Image Creation
- Container Security Scanning
- Image Publishing
- Kubernetes Deployment on Amazon EKS

---

## 🏗️ Architecture

GitHub → Jenkins → SonarQube → OWASP Dependency Check → Docker Build → Trivy Scan → Docker Hub → Amazon EKS → Kubernetes Service

---

## 🛠️ Tools & Technologies

- Git & GitHub
- Jenkins
- SonarQube
- OWASP Dependency Check
- Trivy
- Docker
- Docker Hub
- AWS CLI
- EKSCTL
- Kubernetes
- Amazon EKS
- .NET 6

---

## 📂 Clone Repository

```bash
git clone <repository-url>
cd Dotnet
```

---

## 🐳 Docker Build

Build Docker image:

```bash
docker build -t dotnet:v1 -f build/Dockerfile .
```

Verify:

```bash
docker images
```

---

## ▶️ Run Container

```bash
docker run -d --name dotnet-container -p 5000:5000 dotnet:v1
```

Verify running container:

```bash
docker ps
```

Access application:

```text
http://<Server-Public-IP>:5000
```

---

## 📦 Push Image to Docker Hub

Tag image:

```bash
docker tag dotnet:v1 naliniselv/dotnet:latest
```

Login:

```bash
docker login
```

Push image:

```bash
docker push naliniselv/dotnet:latest
```

---

## 🔍 SonarQube Analysis

SonarQube is integrated into Jenkins to perform:

- Static Code Analysis
- Security Analysis
- Bug Detection
- Code Smell Detection
- Quality Gate Validation

---

## 🛡️ OWASP Dependency Check

Scans third-party dependencies for known vulnerabilities (CVEs).

### Jenkins Stage

```groovy
stage('OWASP Dependency Check') {
    steps {
        dependencyCheck(
            additionalArguments: '--scan ./ --format XML',
            odcInstallation: 'DP-Check'
        )

        dependencyCheckPublisher(
            pattern: '**/dependency-check-report.xml'
        )
    }
}
```

---

## 🔐 Trivy Security Scan

Scans Docker images for:

- Vulnerabilities
- Secrets
- Misconfigurations

### Manual Scan

```bash
trivy image naliniselv/dotnet:latest
```

### Jenkins Stage

```groovy
stage('Trivy Scan') {
    steps {
        sh 'trivy image naliniselv/dotnet:latest'
    }
}
```

---

## ☸️ Amazon EKS Cluster Creation

Create EKS Cluster:

```bash
eksctl create cluster \
--name dotnet-cluster \
--region ap-south-1 \
--node-type t3.medium \
--nodes 2
```

Verify Cluster:

```bash
kubectl get nodes
```

---

## 🚀 Kubernetes Deployment

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dotnet-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dotnet-app
  template:
    metadata:
      labels:
        app: dotnet-app
    spec:
      containers:
      - name: dotnet-app
        image: naliniselv/dotnet:latest
        ports:
        - containerPort: 5000
```

Deploy Application:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deployments
kubectl get pods
```

---

## 🌐 Kubernetes Service

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: dotnet-app-service
spec:
  selector:
    app: dotnet-app
  ports:
    - port: 5000
      targetPort: 5000
  type: LoadBalancer
```

Apply Service:

```bash
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get svc
```

---

## 📊 Monitoring & Troubleshooting

Check Pods:

```bash
kubectl get pods
```

Check Services:

```bash
kubectl get svc
```

View Logs:

```bash
kubectl logs <pod-name>
```

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

---

## 🔄 CI/CD Pipeline Workflow

### Stage 1
Source code pushed to GitHub.

### Stage 2
Jenkins pipeline automatically triggers.

### Stage 3
SonarQube performs code quality and security analysis.

### Stage 4
OWASP Dependency Check scans project dependencies.

### Stage 5
Docker image is built.

### Stage 6
Trivy scans Docker image for vulnerabilities.

### Stage 7
Docker image is pushed to Docker Hub.

### Stage 8
Application is deployed to Amazon EKS.

### Stage 9
Kubernetes Service exposes the application.

---

## ✅ Project Outcome

- Automated CI/CD Pipeline
- Improved Code Quality
- Dependency Vulnerability Scanning
- Container Security Scanning
- Dockerized .NET Application
- Kubernetes Deployment on AWS EKS
- End-to-End DevSecOps Implementation

---

## 👩‍💻 Author

**Nalini Selvaraj**  
Senior Consultant