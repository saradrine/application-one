pipeline {
    agent any

    tools {
        maven 'M3'
        jdk 'jdk17'
        'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker'
    }

    environment {
        DOCKER_IMAGE = 'rymjbeli/application-one'
        VERSION = "${new Date().format('yyyyMMdd-HHmm')}"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        // Add Docker host configuration (for Docker Desktop on Windows)
        DOCKER_HOST = "tcp://host.docker.internal:2375"
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'jenkins-config-v2', url: 'https://github.com/saradrine/application-one.git'
            }
        }

        stage('Verify Docker') {
            steps {
                script {
                    // Verify Docker is available
                    sh '''
                        echo "Checking Docker installation..."
                        docker --version || echo "Docker not found"
                        echo "Docker host: $DOCKER_HOST"
                    '''
                }
            }
        }

        stage('Build with Maven') {
            steps {
                script {
                    try {
                        sh 'mvn --version'
                        sh 'mvn clean package -DskipTests'
                        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                    } catch (e) {
                        error("Build failed: ${e}")
                    }
                }
            }
            post {
                success {
                    stash includes: 'target/*.jar', name: 'app-jar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withTool('docker') {
                        // First build the image
                        sh "docker build -t ${DOCKER_IMAGE}:${VERSION} ."
                        
                        // Then tag it
                        sh "docker tag ${DOCKER_IMAGE}:${VERSION} ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Push vers Docker Hub') {
            steps {
                script {
                    docker.withTool('docker') {
                        withCredentials([usernamePassword(
                            credentialsId: DOCKER_CREDENTIALS_ID,
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh """
                                echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                                docker push ${DOCKER_IMAGE}:${VERSION}
                                docker push ${DOCKER_IMAGE}:latest
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Déploiement terminé avec succès.'
        }
        failure {
            echo 'Échec du pipeline.'
        }
    }
}
