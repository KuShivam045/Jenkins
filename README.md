# Jenkins Pipeline for Deploying Docker Image to AWS Public ECR and Kubernetes

This repository contains a Jenkins pipeline script for building a Docker image, pushing it to an AWS public Elastic Container Registry (ECR), and deploying it to a Kubernetes cluster.

## Prerequisites

Before using this pipeline, ensure that you have the following:

- Jenkins server set up with the necessary plugins (e.g., Docker Pipeline, Git, Kubernetes CLI).
- AWS CLI configured with credentials that have access to your ECR and Kubernetes cluster.
- A public AWS ECR repository.
- A Kubernetes cluster with the required configurations (e.g., kubeconfig, deployment manifest).

## Pipeline Overview

This pipeline performs the following steps:

1. **Checkout Code**: Clones the code from the specified GitHub repository.
2. **Build Docker Image**: Builds a Docker image using the Dockerfile in the repository.
3. **Authenticate to Public AWS ECR**: Authenticates Docker to the AWS public ECR.
4. **Tag and Push Docker Image to Public ECR**: Tags the Docker image and pushes it to the specified public ECR.
5. **Deploy to Kubernetes**: Applies the Kubernetes deployment manifest (`deployment.yml`) to update the image in the Kubernetes cluster.
6. **Cleanup**: Cleans up the Docker images on the Jenkins agent to free up space.

## Jenkins Pipeline Script

```groovy
pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-1'  // Specify your AWS region for public ECR
        ECR_PUBLIC_REGISTRY = 'public.ecr.aws/o8g4t5k9'  // Replace with your public ECR registry URI
        ECR_REPOSITORY = 'k8s-django'  // Replace with your ECR repository name
        IMAGE_TAG = "latest"  // Using latest tag or you can use ${env.BUILD_ID} for dynamic tags
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/KuShivam045/Jenkins.git'   
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ECR_REPOSITORY}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Authenticate to Public AWS ECR') {
            steps {
                script {
                    sh '''
                        aws ecr-public get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_PUBLIC_REGISTRY}
                    '''
                }
            }
        }
        
        stage('Tag and Push Docker Image to Public ECR') {
            steps {
                script {
                    sh '''
                        docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${ECR_PUBLIC_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                    '''
                    sh '''
                        docker push ${ECR_PUBLIC_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                        kubectl apply -f deployment.yml
                    '''
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker rmi ${ECR_REPOSITORY}:${IMAGE_TAG} || true'
            sh 'docker rmi ${ECR_PUBLIC_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} || true'
        }
        success {
            echo 'Deployment to Public AWS ECR was successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
