pipeline {
    agent any

    environment {
        AWS_REGION      = 'us-east-1'
        ECR_REGISTRY    = '493588233491.dkr.ecr.us-east-1.amazonaws.com/jenkishivu'
        ECR_REPO        = 'jenkishivu'
        IMAGE_TAG       = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git credentialsId: 'git-creds',
                    url: 'https://github.com/shivu0129/jenkis-pipe-shiv.git',
                    branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to AWS ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-ecr-creds', region: "${AWS_REGION}") {
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} \
                            | docker login --username AWS \
                            --password-stdin ${ECR_REGISTRY}

                            docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                            docker push ${ECR_REGISTRY}/${ECR_REPO}:latest
                        """
                    }
                }
            }
        }

        stage('Clean Up Local Image') {
            steps {
                sh """
                    docker rmi ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                    docker rmi ${ECR_REGISTRY}/${ECR_REPO}:latest
                """
            }
        }

    }

    post {
        success {
            echo "✅ Image pushed: ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Build Failed — check logs above!"
        }
    }
}
