#!/usr/bin/env groovy
pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "us-east-1"
        DOCKERHUB_ID = credentials('docker_id')
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
        KUBE_CONFIG = credentials('config')
        CREDENTIAL = credentials('CREDENTIALS')
        REPOSITORY_PREFIX= "petcli"
    }
    
    stages {
        stage("Create an EKS Cluster") {
            steps {
                script {
                    dir('terraform') {
                        sh "terraform init -reconfigure"
                        sh "terraform init"
                        sh "terraform destroy -auto-approve"
                    }
                }
            }
        }
        stage("Deploy to EKS") {
            steps {
                script {
                    dir('kubernetes') {
                        sh "aws eks update-kubeconfig --name myapp-eks-cluster"
                        sh "kubectl apply -f nginx-deployment.yaml"
                        sh "kubectl apply -f nginx-service.yaml"
                        echo 'Deploy done'
                    }
                }
            }
        }
    }
}
