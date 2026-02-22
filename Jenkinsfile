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
                sh 'mvn clean package -DskipTests -Dcheckstyle.skip=true'
            }
        }

        stage('SonarQube Analysis') {
    steps {
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
            sh '''
            mvn sonar:sonar \
              -Dsonar.projectKey=petclinic \
              -Dsonar.projectName=petclinic \
              -Dsonar.host.url=http://10.0.0.153:9000 \
              -Dsonar.login=$SONAR_TOKEN
            '''
        }
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
