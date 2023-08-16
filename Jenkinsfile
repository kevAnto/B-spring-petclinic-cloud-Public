#!/usr/bin/env groovy
pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "us-east-1"
        DOCKERHUB_ID = credentials('docker_id')
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
        REPOSITORY_PREFIX= "petcli"
    }
    
    stages {
        stage("Build and Push Images") {
            steps {
              sh '''
                ls ./scripts/ 
                docker login -u $DOCKERHUB_ID -p $DOCKERHUB_PASSWORD
                mvn spring-boot:build-image -Pk8s -DREPOSITORY_PREFIX=$DOCKERHUB_ID 
                docker push $DOCKERHUB_ID/spring-petclinic-cloud-api-gateway:latest
                docker push $DOCKERHUB_ID/spring-petclinic-cloud-visits-service:latest
                docker push $DOCKERHUB_ID/spring-petclinic-cloud-vets-service:latest
                docker push $DOCKERHUB_ID/spring-petclinic-cloud-customers-service:latest
                docker push $DOCKERHUB_ID/spring-petclinic-cloud-admin-server:latest
                docker push $DOCKERHUB_ID/spring-petclinic-cloud-discovery-service:latest
                docker push $DOCKERHUB_ID/spring-petclinic-cloud-config-server:latest
                echo 'Images built'
                '''
            }
        }
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
