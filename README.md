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
                     -Dson
