pipeline {
    agent any
    
    tools {
        maven 'M3'
        jdk 'jdk17'
        'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker'
        nodejs 'node'  // Make sure this is configured in Global Tools
    }

    environment {
        DOCKER_IMAGE = 'rymjbeli/application-one'
        VERSION = "${new Date().format('yyyyMMdd-HHmm')}"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        PROJECT_DIR = 'app'  // Add this if your Node.js code is in a subdirectory
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'jenkins-config-v2', url: 'https://github.com/saradrine/application-one.git'
            }
        }

        stage('Verify Project Structure') {
            steps {
                script {
                    // Check if we're in the right directory
                    sh '''
                        pwd
                        ls -la
                        if [ -d "${PROJECT_DIR}" ]; then
                            cd ${PROJECT_DIR}
                        fi
                        ls -la
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    dir(env.PROJECT_DIR ?: '.') {  // Change to project directory if specified
                        sh 'node --version'
                        sh 'npm --version'
                        sh 'npm install'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    dir(env.PROJECT_DIR ?: '.') {
                        sh 'npm run build'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withTool('docker') {
                        dir(env.PROJECT_DIR ?: '.') {
                            sh "docker build -t ${DOCKER_IMAGE}:${VERSION} ."
                        }
                    }
                }
            }
        }

        stage('Push vers Docker Hub') {
            steps {
                script {
                    docker.withTool('docker') {
                        docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                            sh """
                                docker tag ${DOCKER_IMAGE}:${VERSION} ${DOCKER_IMAGE}:latest
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
            archiveArtifacts artifacts: '**/build/**/*', fingerprint: true
        }
        failure {
            echo 'Échec du pipeline.'
        }
    }
}
