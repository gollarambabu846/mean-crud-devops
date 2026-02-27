pipeline {
    agent any

    environment {
        DOCKER_HUB = "gollarambabu"
        IMAGE_BACKEND = "backend"
        IMAGE_FRONTEND = "frontend"
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

        stage('Run Backend Locally') {
            steps {
                script {
                    // Stop any existing container first
                    sh "docker stop mean-app 2>/dev/null || true"
                    sh "docker rm mean-app 2>/dev/null || true"

                    // Run backend on port 6000 locally
                    sh """
                    docker run -d \
                        --name mean-app \
                        -p 6000:5000 \
                        ${DOCKER_HUB}/${IMAGE_BACKEND}:latest
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
                <body style="background-color: #1A237E; color: white; font-weight: bold; padding: 20px;">
                    <h2>Build Notification</h2>
                    <p><b>Project:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>
                    <p><b>URL:</b> <a style="color: #FFEB3B;" href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
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