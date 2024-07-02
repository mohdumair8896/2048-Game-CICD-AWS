
# 2048 Game on AWS

By doing this project, I have gained hands-on experience with deploying containerized applications on Kubernetes, automating deployments with Jenkins, and enhancing security and code quality with Trivy and SonarQube.
## Objectives

-  Containerize the 2048 Game: Create a Docker image for the 2048 game.
- Deploy on Kubernetes: Deploy the Dockerized application to a Kubernetes      cluster running on AWS EC2 instances.
- Automate with Jenkins: Set up a Jenkins pipeline for continuous integration and deployment.
- Enhance Security and Code Quality: Integrate Trivy for security scanning of Docker images and SonarQube for code quality analysis.
## Features

- Docker: Containerizes the 2048 game application to ensure consistency across different environments.
- Kubernetes: Manages the deployment, scaling, and operation of the containerized application.
- AWS EC2: Provides the infrastructure for running the Kubernetes cluster.
- Jenkins CI/CD: Automates the build, test, and deployment processes.
- Trivy: Scans Docker images for vulnerabilities before deployment.
- SonarQube: Analyzes the codebase for bugs, vulnerabilities, and code smells to ensure high code quality.

## Workflow

- Dockerization: Create a Dockerfile to build a Docker image for the 2048 game.
- Kubernetes Deployment: Define Kubernetes deployment and service YAML files to manage the application.
- Jenkins Pipeline: Configure a Jenkins pipeline to automate the CI/CD process, including building the Docker image, scanning with Trivy, analyzing with SonarQube, and deploying to Kubernetes.
- Security and Quality Checks: Use Trivy for security scans and SonarQube for code quality analysis during the CI/CD pipeline.
- Deployment: Deploy the 2048 game to the Kubernetes cluster on AWS EC2 and access it via a LoadBalancer service.

## Technologies Used

- Docker
- Kubernetes
- AWS EC2
- Jenkins
- Trivy
- SonarQube

## Jenkins Pipeline

```bash
 pipeline{
    agent any
    tools{
        jdk 'java17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'master', url: 'https://github.com/Aj7Ay/2048-React-CICD.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t 2048 ."
                       sh "docker tag 2048 dev87/2048:latest "
                       sh "docker push dev87/2048:latest "
                    }
                }
            }
        }
        stage("Trivy Image Scan"){
            steps{
                sh "trivy image dev87/2048:latest > trivyimagereport.txt"
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name 2048 -p 3000:3000 dev87/2048:latest'
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f deployment.yaml'
                  }
                }
            }
        }
    }
}
```


## Screenshots

### First Build was successful
![App Screenshot](https://github.com/mohdumair8896/2048-Game-CICD-AWS/blob/master/Screenshots%20of%20Project/Screenshot%20from%202024-07-03%2002-07-20.png)

### SonarQube Quality Passed
![App Screenshot](https://github.com/mohdumair8896/2048-Game-CICD-AWS/blob/master/Screenshots%20of%20Project/Screenshot%20from%202024-07-03%2002-08-33.png)

### Image pushed to DockerHub
![App Screenshot](https://github.com/mohdumair8896/2048-Game-CICD-AWS/blob/master/Screenshots%20of%20Project/Screenshot%20from%202024-07-03%2002-44-40.png)

### 3 Instances Are Running(1-->Jenkins-Server, 2--> Kubernetes (master & slave)
![App Screenshot](https://github.com/mohdumair8896/2048-Game-CICD-AWS/blob/master/Screenshots%20of%20Project/Screenshot%20from%202024-07-03%2003-05-27.png)

### Stage view
![App Screenshot](https://github.com/mohdumair8896/2048-Game-CICD-AWS/blob/master/Screenshots%20of%20Project/Screenshot%20from%202024-07-03%2002-59-51.png)

### Deployed in Kubernetes
![App Screenshot](https://github.com/mohdumair8896/2048-Game-CICD-AWS/blob/master/Screenshots%20of%20Project/Screenshot%20from%202024-07-03%2004-04-53.png)
