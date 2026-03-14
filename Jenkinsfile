pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "277528343142.dkr.ecr.ap-south-1.amazonaws.com/my-first-devops-project"
        TARGET_SERVER = "ubuntu@10.10.11.25"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t demo-app .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin 277528343142.dkr.ecr.ap-south-1.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag demo-app:latest $ECR_REPO:latest'
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh 'docker push $ECR_REPO:latest'
            }
        }

        stage('Deploy to Target EC2') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $TARGET_SERVER '
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin 277528343142.dkr.ecr.ap-south-1.amazonaws.com &&
                    docker pull $ECR_REPO:latest &&
                    docker stop demo-container || true &&
                    docker rm demo-container || true &&
                    docker run -d -p 80:8080 --name demo-container $ECR_REPO:latest
                    '
                    """
                }
            }
        }

    }
}
