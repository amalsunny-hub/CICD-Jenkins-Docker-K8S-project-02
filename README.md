🚀 CI/CD Pipeline with Jenkins, Docker & Kubernetes (Minikube)

This project demonstrates a complete end-to-end CI/CD pipeline where code pushed to GitHub automatically triggers Jenkins to:

Run unit tests

Build a Docker image

Push the image to Docker Hub

Deploy the application to Kubernetes using Minikube

🧱 Tech Stack

GitHub – Source code repository

Jenkins (Multibranch Pipeline) – CI/CD orchestration

Docker – Containerization

Docker Hub – Image registry

Kubernetes (Minikube) – Container orchestration

Python + Flask – Sample application

📁 Project Structure
ci-cd-k8s-project-02/
```
│
├── app.py
├── test_app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
└── k8s/
    ├── deployment.yaml
    └── service.yaml
```
✅ Prerequisites (Already Installed)

⚠️ Installation steps are intentionally excluded.

Git

Jenkins (running on Windows)

Docker Desktop (running)

Minikube (running)

kubectl configured

Python & pip

Docker Hub account

🔹 Step 1: GitHub Repository Setup

Created a GitHub repository

Pushed application source code

Added a Jenkinsfile at the root of the repo

Ensured only one branch (main) exists

Set main as the default branch

🔹 Step 2: Jenkins Configuration
2.1 Create Jenkins Job

Job Type: Multibranch Pipeline

Source: GitHub repository URL

Jenkins automatically detects Jenkinsfile

🔑 Multibranch Pipeline is required for reliable GitHub webhook triggering

2.2 GitHub Webhook

Webhook URL:
```
https://<ngrok-url>/github-webhook/
```

Event: Push

Content type: application/json

✔ Verified webhook delivery (HTTP 200)

2.3 Jenkins Credentials

Created Docker Hub credentials in Jenkins:

Kind: Username with password

ID: dockerhub-creds

Username: Docker Hub username

Password: Docker Hub password / token

⚠️ Jenkinsfile must reference the exact credential ID

🔹 Step 3: Jenkinsfile (Pipeline Logic)
Pipeline Flow

Generate Docker image tag from Git commit

Install Python dependencies

Run unit tests using pytest

Build Docker image

Push image to Docker Hub

Deploy application to Kubernetes

Jenkinsfile (Final Working Version)
```
pipeline {
    agent any

    environment {
        IMAGE_NAME = "amalsunny27/ci-cd-k8s-project-02"
    }

    stages {

        stage('Set Image Tag') {
            steps {
                script {
                    env.TAG = env.GIT_COMMIT.substring(0, 7)
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Run Unit Tests') {
            steps {
                bat 'pytest test_app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%TAG% .'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    bat '''
                        docker login -u %DOCKERHUB_USERNAME% -p %DOCKERHUB_PASSWORD%
                        docker push %IMAGE_NAME%:%TAG%
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f k8s'
            }
        }
    }
}
```
🔹 Step 4: Dockerfile
```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```
🔹 Step 5: Kubernetes Manifests
Deployment (k8s/deployment.yaml)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app-container
          image: amalsunny27/ci-cd-k8s-project-02:latest
          ports:
            - containerPort: 5000
```
```
Service (k8s/service.yaml)
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30007
```
🔹 Step 6: Deployment Verification
```
kubectl get pods
kubectl get svc
```

Access application:
```
minikube service my-app-service
```
🛠 Key Issues Faced & Fixes (IMPORTANT)
Issue	Fix
Jenkins webhook not triggering	Switched from Pipeline → Multibranch Pipeline
Branch mismatch (master vs main)	Unified to single main branch
sh not found error	Replaced sh with bat (Windows Jenkins)
$VAR not working	Used %VAR% for Windows
Docker credential error	Fixed credentialsId mismatch
Kubernetes apiVersion not set	Fixed YAML key typos & casing
nodePort error	Added type: NodePort
🎯 Final Outcome

✔ GitHub push automatically triggers Jenkins
✔ Jenkins runs tests and builds Docker image
✔ Image is pushed to Docker Hub
✔ Application is deployed to Kubernetes (Minikube)

This project represents a real-world CI/CD pipeline, including debugging, fixes, and best practices.
