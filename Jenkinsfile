pipeline {
    agent any

    environment {
        DOCKER_HUB = "gollarambabu"
        IMAGE_BACKEND = "backend"
        IMAGE_FRONTEND = "frontend"
        VM_IP = "65.2.99.12"
        SONAR_SERVER_NAME = "SonarQube"
    }

    tools {
        nodejs "nodejs"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/gollarambabu846/mean-crud-devops.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube'

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=mean-app \
                          -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Skipping Quality Gate temporarily"
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan .',
                                odcInstallation: 'OWASP-Dependency-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh "docker build -t ${DOCKER_HUB}/${IMAGE_BACKEND}:latest ./backend"
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh "docker build -t ${DOCKER_HUB}/${IMAGE_FRONTEND}:latest ./frontend"
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker') {
                        sh "docker push ${DOCKER_HUB}/${IMAGE_BACKEND}:latest"
                        sh "docker push ${DOCKER_HUB}/${IMAGE_FRONTEND}:latest"
                    }
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                sshagent(['vm-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@65.2.99.12 "
                        docker pull ${DOCKER_HUB}/${IMAGE_BACKEND}:latest &&
                        docker stop mean-app || true &&
                        docker rm mean-app || true &&
                        docker run -d --name mean-app -p 80:3000 ${DOCKER_HUB}/${IMAGE_BACKEND}:latest
                        "
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully 🚀"
        }
        failure {
            echo "Pipeline failed ❌ Check logs"
        }
    }
}