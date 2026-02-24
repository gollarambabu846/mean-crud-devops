pipeline {
    agent any

    environment {
        DOCKER_HUB = "gollarambabu"
        IMAGE_BACKEND = "backend"
        IMAGE_FRONTEND = "frontend"
        VM_IP = "YOUR_VM_IP"
        SONARQUBE_ENV = "SonarQube"
    }

    tools {
        nodejs "nodejs"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/<your-username>/mean-crud-devops.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh """
                    sonar-scanner \
                      -Dsonar.projectKey=mean-app \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://$VM_IP:9000 \
                      -Dsonar.login=${SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
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
                sh "docker build -t $DOCKER_HUB/$IMAGE_BACKEND:latest ./backend"
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh "docker build -t $DOCKER_HUB/$IMAGE_FRONTEND:latest ./frontend"
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                usernameVariable: 'DOCKER_USER',
                                passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_HUB/$IMAGE_BACKEND:latest
                    docker push $DOCKER_HUB/$IMAGE_FRONTEND:latest
                    """
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                sshagent(['vm-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@$VM_IP '
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