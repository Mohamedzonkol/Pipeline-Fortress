# CI/CD Pipeline Project with Kubernetes

## Overview
This project is a complete CI/CD pipeline built using Jenkins, SonarQube, Nexus, Trivy, Docker, and Kubernetes. The pipeline automates code compilation, testing, security scanning, quality checks, and deployment to a Kubernetes cluster. Monitoring is implemented using Prometheus and Grafana.

## Architecture
The pipeline architecture follows these steps:

1. **Source Code Management:**
   - Developers push code to GitHub.
   
2. **Continuous Integration:**
   - Jenkins triggers the pipeline upon detecting changes in the GitHub repository.
   - The code is compiled and unit tests are executed using Maven.
   - Trivy is used to perform a vulnerability scan on the codebase.

3. **Code Quality and Security:**
   - SonarQube performs code quality analysis.
   - The pipeline waits for SonarQube’s quality gate before proceeding.
   - KubeAudit performs a security scan on the Kubernetes cluster.

4. **Artifact Management:**
   - The application is packaged using Maven.
   - The package is stored in Nexus Repository.

5. **Dockerization and Container Security:**
   - Docker builds and tags the image.
   - Trivy performs a Docker image scan.
   - The Docker image is pushed to Docker Hub.

6. **Continuous Deployment:**
   - The application is deployed to a Kubernetes cluster.
   - The deployment is verified by checking the resources like Pods and Services on the cluster.

7. **Monitoring:**
   - Prometheus and Grafana are used to monitor the application and server metrics.

## Pipeline Diagram

![CI/CD Pipeline](pipeline.jpeg)

## Project Setup

### 1. Create Virtual Machines
- **Master Node:** Create a VM on Azure to act as the Kubernetes master node.
- **Worker Node:** Create another VM on Azure for the worker nodes.

### 2. Kubernetes Cluster Setup
- **Install Container Runtime:** Install a container runtime (e.g., Docker) on both nodes.
- **Install Kubeadm, Kubelet, and Kubectl:** These tools are required for managing the Kubernetes cluster.
- **Initialize the Kubernetes Cluster:** Use `kubeadm` to initialize the master node.
- **Join Worker Nodes to the Cluster:** Use `kubeadm` join command to add worker nodes to the master.

### 3. Security and Quality Tools Installation
- **Install KubeAudit:** Scan the Kubernetes cluster for security issues.
- **Install SonarQube:** Deploy SonarQube using Docker for code quality analysis.
- **Install Nexus:** Deploy Nexus using Docker for artifact management.

### 4. Jenkins Setup
- **Install Jenkins:** Create a VM for Jenkins, install it, and configure the necessary plugins.
- **Setup Credentials and Tools:** Add credentials for GitHub, Docker, Kubernetes, and Nexus in Jenkins. Install required tools such as Maven, JDK, and SonarQube scanner.

### 5. Build and Deploy Pipeline
- **Pipeline Script:** The Jenkins pipeline is defined as follows:

  ```groovy
  pipeline {
      agent any
      tools {
          jdk 'jdk17'
          maven 'maven-3'
      }
      environment {
          SCANNER_HOME=tool 'sonar-scanner'
      }
      stages {
          stage('Checkout the Project') {
              steps {
                  git branch: 'main', credentialsId: 'git-credintial', url: 'https://github.com/Mohamedzonkol/CICD-Corporate-DevOps-Pipeline-Project.git'
              }
          }
          stage('Compile the Source Code') {
              steps {
                  sh "mvn compile"
              }
          }
          stage('Test unit test cases') {
              steps {
                  sh "mvn test"
              }
          }
           stage('Vulnerability Scan by Trivy') {
              steps {
                  sh "trivy fs --format table -o trivy-fs-report.html ."
              }
          }
          stage('SonarQube Analysis') {
              steps {
                  withSonarQubeEnv('sonar') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=projecttest -Dsonar.projectKey=projecttest \
                     -Dsonar.java.binaries=. '''
                  }
              }
          }
          stage('Wait for Quality Gate') {
              steps {
                  script {
                      waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                  }
              }
          }
          stage('Build the Package') {
              steps {
                  sh "mvn package"
              }
          }
          stage ('Build & Tag the Docker Image') {
              steps {
                  script {
                      withDockerRegistry(credentialsId: 'dock-cred', toolName: 'docker') {
                          sh "docker build -t mohamedelsayed7/techgame:latest ."
                      }
                  }
              }
          }
          stage('Docker Image Scanning by Trivy') {
              steps {
                  sh "trivy image --format table -o trivy-image-report.html mohamedelsayed7/techgame"
              }
          }
          stage ('Push the Docker Image to Docker Hub') {
              steps {
                  script {
                      withDockerRegistry(credentialsId: 'dock-cred', toolName: 'docker') {
                          sh "docker push mohamedelsayed7/techgame:latest"
                      }
                  }
              }
          }
          stage ('Deploy the Docker Image to k8s Cluster') {
              steps {
                  withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://50.642.15.22:6443') {
                      sh '''kubectl apply -f deployment.yml
                            kubectl apply -f service.yml
                      '''
                  }
              }
          }
          stage ('Verify the Resources like POD, SVC on k8s cluster') {
              steps {
                  withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://50.642.15.22:6443') {
                      sh 'kubectl get pods -n webapps'
                      sh 'kubectl get svc -n webapps'
                  }
              }
          }
      }
  }
## Monitoring Setup

- **Prometheus:** Install Prometheus to collect metrics from Kubernetes and your applications. Configure it to scrape data from your Kubernetes nodes, pods, and any other endpoints that you wish to monitor.
  
- **Grafana:** Install Grafana and configure it to use Prometheus as a data source. Create dashboards in Grafana to visualize the metrics collected by Prometheus, including resource usage, application performance, and infrastructure health.

- **Blackbox Exporter:** Install Blackbox Exporter to monitor the availability of your services and endpoints. This tool allows you to perform health checks (e.g., HTTP, DNS, TCP) on your services and visualize the results in Grafana.

- **Node Exporter:** Deploy Node Exporter on all VMs to monitor hardware and OS metrics such as CPU, memory, and disk usage. Prometheus will collect these metrics and Grafana can be used to visualize them.
## Conclusion
This pipeline automates the entire CI/CD process, from code integration and testing to deployment and monitoring in a Kubernetes environment. It ensures that the code is secure, high-quality, and always ready for production.
