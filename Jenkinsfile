pipeline {
    agent any
    tools {
        maven 'Maven3'
    }
    environment {
        ECR_URI = '141884504048.dkr.ecr.eu-north-1.amazonaws.com/petclinic-repo'
        REGION = 'eu-north-1'
        IMAGE_TAG = 'latest'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build JAR') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                withAWS(credentials: 'aws-p1', region: "${REGION}") {
                    sh '''
                    aws ecr get-login-password --region ${REGION} | \
                    docker login --username AWS --password-stdin ${ECR_URI}

                    docker build -t ${ECR_URI}:${IMAGE_TAG} .

                    docker push ${ECR_URI}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage('Deploy to ECS') {
            steps {
                withAWS(credentials: 'aws-p1', region: "${REGION}") {
                    sh '''
                    aws ecs update-service \
                      --cluster petclinic-cluster \
                      --service petclinic-service \
                      --force-new-deployment \
                      --region ${REGION}
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Petclinic deployed successfully!'
        }
    }
}
