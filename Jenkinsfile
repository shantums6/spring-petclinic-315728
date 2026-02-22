pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ACCOUNT_ID = "443112456205"
        IMAGE_NAME = "petclinc-app"
        REPO_NAME = "petclinic"
    }

    stages {

        stage('Build with Maven') {
            tools {
        maven 'maven3'
    }
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'sudo docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                sudo docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                sudo docker tag $IMAGE_NAME:latest \
                $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                sudo docker push \
                $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:latest
                '''
            }
        }
    }
}
