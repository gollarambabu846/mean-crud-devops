pipeline {
    agent any

    environment {
        DOCKER_HUB = "gollarambabu"
        IMAGE_BACKEND = "backend"
        IMAGE_FRONTEND = "frontend"
        VM_IP = "13.235.42.120"
        // Ensure this matches the Name in Manage Jenkins > System > SonarQube servers
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
                    // This matches the Name in Manage Jenkins > Tools > SonarQube Scanner
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
                timeout(time: 5, unit: 'MINUTES') {
                    // Note: This requires a Webhook configured in SonarQube
                    waitForQualityGate abortPipeline: true
                }
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
                    // 'docker' is the Credentials ID for your Docker Hub login
                    docker.withRegistry('https://index.docker.io/v1/', 'docker') {
                        sh "docker push ${DOCKER_HUB}/${IMAGE_BACKEND}:latest"
                        sh "docker push ${DOCKER_HUB}/${IMAGE_FRONTEND}:latest"
                    }
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                // 'vm-ssh-key' is the Credentials ID for your VM private key
                sshagent(['vm-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${VM_IP} '
                    cd mean-crud-devops &&
                    docker-compose pull &&
                    docker-compose up -d
                    '
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