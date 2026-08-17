pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '377730029945'
        ECR_REPO = 'tech-challenge-2-app'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_URI = "${ECR_REGISTRY}/${ECR_REPO}:latest"
        EKS_CLUSTER = 'tc2-eks-cluster'
        HELM_RELEASE = 'tc2-helm'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${ECR_REPO}:latest .
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag ${ECR_REPO}:latest ${IMAGE_URI}
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                    docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Configure kubectl') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                      --name ${EKS_CLUSTER} \
                      --region ${AWS_REGION}
                '''
            }
        }

        stage('Verify EKS Access') {
            steps {
                sh '''
                    kubectl get nodes
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    helm upgrade --install ${HELM_RELEASE} ./helm/tc2-app \
                      --set image.repository=${ECR_REGISTRY}/${ECR_REPO} \
                      --set image.tag=latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployments
                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo 'Tech Challenge 2 deployment completed successfully.'
        }

        failure {
            echo 'Tech Challenge 2 deployment failed. Review the Jenkins console output.'
        }
    }
}