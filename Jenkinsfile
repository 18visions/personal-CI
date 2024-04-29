pipeline {
    agent any

    environment {
        IMAGE_NAME = 'your-image-name'
        ECR_REPOSITORY = 'your-ecr-repository-name'
        AWS_REGION = 'your-aws-region'
        AWS_ACCOUNT_ID = 'your-aws-account-id'
        VERSION = '1.0.0'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Prepare ECR') {
            steps {
                script {
                    // Check if the ECR repository exists and create it if it does not
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_CREDENTIALS_ID', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        sh """
                        aws ecr describe-repositories --repository-names ${env.ECR_REPOSITORY} --region ${env.AWS_REGION} || \
                        aws ecr create-repository --repository-name ${env.ECR_REPOSITORY} --region ${env.AWS_REGION}
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPOSITORY}:${env.VERSION} ."
                }
            }
        }

        stage('Push to AWS ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_CREDENTIALS_ID', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        // Login to AWS ECR and push the image
                        sh """
                        aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com
                        docker push ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPOSITORY}:${env.VERSION}
                        """
                    }
                }
            }
        }

        stage('Clean Up') {
            steps {
                script {
                    // Remove the local Docker images to free up space
                    sh "docker rmi ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPOSITORY}:${env.VERSION}"
                }
            }
        }
    }

    post {
        always {
            // Steps to clean up, notify, etc.
        }
    }
}
