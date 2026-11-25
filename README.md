AWS-Based Automated CI/CD Pipeline Using Jenkins, Docker, GitHub & Docker Hub

1. Project Overview

This project demonstrates an automated CI/CD pipeline built on AWS EC2 using Jenkins, Docker, GitHub, and Docker Hub.

Whenever a developer pushes code changes to GitHub, Jenkins automatically detects the changes, builds a Docker image, pushes the image to Docker Hub, and deploys the updated application on the EC2 instance.

Cloud Provider: AWS
Operating System: Amazon Linux
Application: Python Flask
Containerization: Docker

2. Architecture

        Developer
            |
            v
       GitHub Repository
            |
            v
       Jenkins on EC2
            |
            v
       Docker Build
            |
            v
       Docker Hub Push
            |
            v
       Deploy on AWS EC2
            |
            v
       Flask Application
        Port: 5000

Jenkins uses Poll SCM to check the GitHub repository every two minutes.

3. AWS Infrastructure

Resource| Configuration
Cloud Provider| AWS
Compute| EC2
Instance Type| t3.small
Operating System| Amazon Linux
VPC| devops-cicd-vpc
Public Subnet| devops-cicd-public-subnet
Internet Gateway| devops-cicd-igw
Route Table| devops-cicd-public-rt
Security Group| devops-cicd-sg

Security Group Ports

Port| Service
22| SSH
8080| Jenkins
5000| Python Flask Application

Restrict SSH and Jenkins access to trusted IP addresses.

4. Technologies Used

- AWS EC2
- Amazon Linux
- Jenkins
- Java 21
- Docker
- Git
- GitHub
- Docker Hub
- Python
- Flask
- Linux Shell Commands
- Jenkins Pipeline (Groovy)

5. Application Structure

aws-devops-cicd-app/
│
├── app.py
├── requirements.txt
├── Dockerfile
└── README.md

app.py

from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello! My First Automatic AWS CI/CD Deployment!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

requirements.txt

Flask==3.1.2

Dockerfile

FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]

6. Jenkins CI/CD Pipeline

The Jenkins pipeline contains four stages:

1. GitHub Checkout: Fetches application source code.
2. Docker Build: Creates a Docker image using Jenkins build number.
3. Docker Hub Push: Pushes versioned and latest images to Docker Hub.
4. Deploy: Stops the previous application container and starts the new version.

Jenkinsfile

pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('GitHub Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/tusharsinha05/aws-devops-cicd-app.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t aws-devops-cicd-app:${BUILD_NUMBER} .
                    docker tag aws-devops-cicd-app:${BUILD_NUMBER} aws-devops-cicd-app:latest
                '''
            }
        }

        stage('Docker Hub Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker tag aws-devops-cicd-app:${BUILD_NUMBER} $DOCKER_USER/aws-devops-cicd-app:${BUILD_NUMBER}
                        docker tag aws-devops-cicd-app:${BUILD_NUMBER} $DOCKER_USER/aws-devops-cicd-app:latest

                        docker push $DOCKER_USER/aws-devops-cicd-app:${BUILD_NUMBER}
                        docker push $DOCKER_USER/aws-devops-cicd-app:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker stop aws-devops-container || true
                    docker rm aws-devops-container || true

                    docker run -d \
                        --name aws-devops-container \
                        -p 5000:5000 \
                        aws-devops-cicd-app:${BUILD_NUMBER}
                '''
            }
        }
    }
}

Docker Hub Repository:

https://hub.docker.com/r/tusharsinha05/aws-devops-cicd-app

7. Automatic CI/CD Workflow

1. Developer updates application code.
2. Developer commits changes to GitHub.
3. Jenkins checks GitHub using Poll SCM.
4. Jenkins checks out the latest source code.
5. Docker builds a new versioned image.
6. Jenkins pushes the image to Docker Hub.
7. Jenkins deploys the new image on EC2.
8. Application becomes available on port 5000.

8. Verification Commands

Check Docker containers:

sudo docker ps

Check Docker images:

sudo docker images

Check application response:

curl http://localhost:5000

Expected response:

Hello! My First Automatic AWS CI/CD Deployment!

9. Project Outcome

Successfully implemented an automated CI/CD pipeline using AWS EC2, Jenkins, Docker, GitHub, and Docker Hub.

The project demonstrates:

- Automated source code checkout.
- Docker image creation and versioning.
- Docker Hub image publishing.
- Automated application deployment.
- GitHub change detection through Jenkins Poll SCM.
- Application verification on AWS EC2.

10. Future Improvements

- Add automated testing before deployment.
- Implement application health checks.
- Configure HTTPS using a domain and SSL certificate.
- Add monitoring and logging.
- Implement secure deployment practices.
- Introduce automated rollback.
ject: AWS Automated CI/CD Pipeline

GitHub: https://github.com/tusharsinha05/aws-devops-cicd-app
