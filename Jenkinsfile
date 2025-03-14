pipeline {
    agent {
        label 'agent'
    }

    environment {
        AWS_REGION = "sa-east-1"  // Thay thế bằng vùng AWS của bạn
        ECR_REGISTRY = "774305578623.dkr.ecr.${AWS_REGION}.amazonaws.com"
        EKS_CLUSTER = "my-eks-cluster"
        SERVICES = "database-service backend-service api-gateway ui-service"
        GIT_REPO = "https://github.com/Zudypubg/micro-service.git"
        GIT_CREDENTIALS_ID = "GitHub-PAT-Full-Access-4"
    }

    stages {
        // stage('Clone Repository') {
        //     steps {
        //         script {
        //             echo "Cloning GitHub repository..."
        //             git credentialsId: "${GIT_CREDENTIALS_ID}", url: "${GIT_REPO}"
        //         }
        //     }
        // }
        stage('Build Images') {
            steps {
                script {
                    for (service in SERVICES.split(" ")) {
                        dir(service) {
                            echo "Building Docker image for ${service}..."
                            sh """
                            docker build -t ${ECR_REGISTRY}/${service}:latest .
                            """
                        }
                    }
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh """
                    echo "Logging in to ECR..."
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('Push Images to ECR') {
            steps {
                script {
                    for (service in SERVICES.split(" ")) {
                        dir(service) {
                            echo "Pushing Docker image for ${service} to ECR..."
                            sh """
                            docker push ${ECR_REGISTRY}/${service}:latest
                            """
                        }
                    }
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    sh """
                    echo "Configuring kubectl..."
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
                    """
                    
                    for (service in SERVICES.split(" ")) {
                        dir(service) {
                            echo "Applying Kubernetes manifests for ${service}..."
                            sh """
                            kubectl apply -f deployment.yml
                            """
                        }
                    }
                }
            }
        }

        // stage('Verify Deployment') {
        //     steps {
        //         script {
        //             sh """
        //             echo "Checking Kubernetes Pods..."
        //             kubectl get pods -n default
        //             """
        //         }
        //     }
        // }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
