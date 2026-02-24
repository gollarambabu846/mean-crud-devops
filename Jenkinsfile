pipeline {
    agent any

    environment {
        DOCKER_HUB = "gollarambabu"
        IMAGE_BACKEND = "backend"
        IMAGE_FRONTEND = "frontend"
        VM_IP = "13.235.42.120"
        SONAR_SERVER = "sonar-server"
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
            // This pulls the scanner you just configured in Step 1
            def scannerHome = tool 'SonarQube' 

            // This pulls the server details you configured in Step 2
            withSonarQubeEnv('sonar-server') { 
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
                    // This waits for the webhook result from SonarQube
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
                    // 'docker' here is the Credentials ID stored in Jenkins
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