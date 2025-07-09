pipeline {
    agent any

    tools {
        nodejs 'node24'
        jdk 'Java21'
    }

    environment {
        SCANNER_HOME = tool 'SonarScanner'
        AWS_ACCOUNT_ID = '231778609415'
        ECR_REPO_NAME = 'manikiran/amazonprime'
        AWS_REGION = 'us-east-1'
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
    }

    stages {
        stage('1. Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/manikiran7/DevopsProject2.git'
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                withSonarQubeEnv('mysonarqube') {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=amazon-prime \
                        -Dsonar.projectKey=amazon-prime
                    """
                }
            }
        }

        stage('3. Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }

        stage('4. Install npm') {
            steps {
                sh "npm install"
            }
        }

        stage('5. Trivy Scan') {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }

        stage('6. Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO_NAME} ."
            }
        }

        stage('7. Create ECR Repo') {
            steps {
                sh """
                    aws ecr describe-repositories --repository-names ${ECR_REPO_NAME} --region ${AWS_REGION} || \
                    aws ecr create-repository --repository-name ${ECR_REPO_NAME} --region ${AWS_REGION}
                """
            }
        }

        stage('8. Login to ECR & Tag Image') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker tag ${ECR_REPO_NAME} ${ECR_URI}:${BUILD_NUMBER}
                    docker tag ${ECR_REPO_NAME} ${ECR_URI}:latest
                """
            }
        }

        stage('9. Push Image to ECR') {
            steps {
                sh """
                    docker push ${ECR_URI}:${BUILD_NUMBER}
                    docker push ${ECR_URI}:latest
                """
            }
        }

        stage('10. Cleanup Images') {
            steps {
                sh """
                    docker rmi ${ECR_URI}:${BUILD_NUMBER} || true
                    docker rmi ${ECR_URI}:latest || true
                    docker images
                """
            }
        }
    }
}
