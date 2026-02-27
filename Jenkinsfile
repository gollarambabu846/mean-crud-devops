pipeline {
    agent any

    environment {
        DOCKER_HUB = "gollarambabu"
        IMAGE_BACKEND = "backend"
        IMAGE_FRONTEND = "frontend"
        VM_IP = "13.201.58.235"
        SONAR_SERVER_NAME = "sonar"
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
                    def scannerHome = tool 'sonar-scanner'

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
                        ssh -o StrictHostKeyChecking=no ubuntu@${VM_IP} "
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
        always {
            emailext(
                attachLog: true,
                subject: "${currentBuild.currentResult}: ${env.JOB_NAME}",
                body: """
                <html>
                <body>
                    <h2>Build Notification</h2>
                    <p><b>Project:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>
                    <p><b>URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                </body>
                </html>
                """,
                to: 'gollarambabu70@gmail.com',
                mimeType: 'text/html'
            )
        }

        success {
            echo "Pipeline executed successfully 🚀"
        }

        failure {
            echo "Pipeline failed ❌ Check logs"
        }
    }
}