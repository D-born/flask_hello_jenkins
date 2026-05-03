# Jenkins - Docker - Kubernetes Mini Project

## Overview

This project demonstrates a complete CI/CD workflow built around:

- Jenkins
- Docker
- Kubernetes
- a Python Flask application

The objective is to automate testing and deployment of a simple web application using Jenkins pipelines and Kubernetes agents.

## Project Structure

```text
flask_app/
├── app.py
├── test.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── values.yaml
├── kubernetes/
│   ├── deployment.yaml
│   └── service.yaml
└── README.md
```

## Application

The application is a simple Flask service exposing:

- `/`
- `/hello/`
- `/hello/<username>`
- `/feature/<username>`

Unit tests are implemented with Python `unittest`.

## Completed Work

The project includes the following completed steps:

1. Jenkins installation with Docker
2. Creation of a first Jenkins pipeline (`hello`)
3. Development of the Flask application
4. Local execution of unit tests
5. Docker image creation and validation
6. Local Docker registry usage
7. GitHub repository integration
8. Jenkins deployment on Kubernetes
9. Pipeline as code with `Jenkinsfile`
10. Kubernetes deployment manifests
11. TDD workflow on branch `feature1`

## Local Test Execution

From the project directory:

```powershell
cd C:\Users\kenny\Desktop\tp_jenkins\flask_app
.\.venv\Scripts\python.exe test.py -v
```

Expected result:

```text
Ran 4 tests ...
OK
```

## Docker Build

Build the application image:

```powershell
cd C:\Users\kenny\Desktop\tp_jenkins\flask_app
docker build -t flask_hello .
```

Run tests inside the container:

```powershell
docker run --rm flask_hello ./test.py -v
```

## Local Registry

Start the local registry:

```powershell
docker run -d -p 4000:5000 --name registry registry
```

Tag and push the image:

```powershell
docker tag flask_hello localhost:4000/flask_hello
docker push localhost:4000/flask_hello
```

## Jenkins with Docker

Jenkins was first started from the `jenkins_simple` directory using Docker Compose.

Typical commands:

```powershell
cd C:\Users\kenny\Desktop\tp_jenkins\jenkins_simple
docker compose up -d
docker compose logs
```

## Jenkins on Kubernetes

Jenkins was deployed in Kubernetes using Helm.

Typical setup commands:

```powershell
kubectl create namespace jenkins
kubectl config set-context --current --namespace=jenkins
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm show values jenkins/jenkins > values.yaml
helm install jenkins jenkins/jenkins -n jenkins -f values.yaml
```

Access can be established locally with:

```powershell
kubectl port-forward svc/jenkins 32000:8080 -n jenkins
```

Then open:

```text
http://localhost:32000
```

## Jenkins Pipeline

The Jenkins pipeline is defined in:

- [Jenkinsfile](C:/Users/kenny/Desktop/tp_jenkins/flask_app/Jenkinsfile)

The pipeline covers:

- source checkout
- Python dependency installation
- test execution
- Kubernetes deployment

## Kubernetes Deployment

Deployment files:

- [kubernetes/deployment.yaml](C:/Users/kenny/Desktop/tp_jenkins/flask_app/kubernetes/deployment.yaml)
- [kubernetes/service.yaml](C:/Users/kenny/Desktop/tp_jenkins/flask_app/kubernetes/service.yaml)

Apply manually if needed:

```powershell
kubectl apply -f .\kubernetes\deployment.yaml -n jenkins
kubectl apply -f .\kubernetes\service.yaml -n jenkins
```

Check cluster resources:

```powershell
kubectl get deployment -n jenkins
kubectl get pods -n jenkins
kubectl get svc -n jenkins
```

## TDD Workflow

The TDD part was completed on branch `feature1`.

Main steps:

1. Create the feature branch
2. Add a failing test for `/feature/<username>`
3. Push the failing test
4. Implement the missing Flask route
5. Re-run tests and push the fix

Recent commits include:

- `Add failing feature route test`
- `Implement feature route`
- `Adjust deployment image for Kubernetes`

## Git Status

Useful commands:

```powershell
git status
git branch -vv
git log --oneline -5
```

## Final Result

This repository contains a working educational CI/CD project demonstrating:

- Jenkins setup
- Docker image build workflow
- Kubernetes-based Jenkins agents
- application deployment to Kubernetes
- a simple TDD workflow using Git branches
