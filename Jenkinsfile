pipeline {
    agent any  // Using 'any' instead of docker agent to avoid conflicts
    
    tools {
        nodejs 'nodejs'  // Configure Node.js in Global Tools
    }

    environment {
        DOCKER_IMAGE = 'rymjbeli/application-one'
        VERSION = "${new Date().format('yyyyMMdd-HHmm')}"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'jenkins-config-v2', url: 'https://github.com/saradrine/application-one.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'node --version'
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Ensure Docker is available
                    sh 'docker --version || echo "Docker not found rim"'
                    
                    // Build with Docker
                    docker.withTool('docker') {
                        docker.build("${DOCKER_IMAGE}:${VERSION}")
                    }
                }
            }
        }

        stage('Push vers Docker Hub') {
            steps {
                script {
                    docker.withTool('docker') {
                        docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                            def builtImage = docker.image("${DOCKER_IMAGE}:${VERSION}")
                            builtImage.push()
                            builtImage.push('latest')  // Also push as latest
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Déploiement terminé avec succès.'
            archiveArtifacts artifacts: 'build/**/*', fingerprint: true
        }
        failure {
            echo 'Échec du pipeline.'
        }
    }
}
