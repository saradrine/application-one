pipeline {
    agent any

    tools {
        maven 'M3'
        jdk 'jdk17'
    }

    environment {
        // Configuration Docker Hub
        DOCKER_HUB = credentials('docker-hub-credentials')
        IMAGE_NAME = 'sara12308/application-one'
        VERSION = "${env.BUILD_NUMBER}"
        BUILD_DATE = new Date().format('yyyyMMdd-HHmmss')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                sh 'git --version'
                echo "Code checked out from ${env.GIT_URL}"
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
                        echo "Build failed: ${e}"
                        currentBuild.result = 'FAILURE'
                        error('Maven build failed')
                    }
                }
            }

            post {
                success {
                    echo 'Maven build completed successfully!'
                    stash includes: 'target/*.jar', name: 'app-jar'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        // Vérifie que Docker fonctionne
                        sh 'docker --version'
                        docker.build("${IMAGE_NAME}:${VERSION}-${BUILD_DATE}")
                    } catch (e) {
                        echo "Docker build failed: ${e}"
                        currentBuild.result = 'FAILURE'
                        error('Docker build failed')
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    try {
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                            docker.image("${IMAGE_NAME}:${VERSION}").push()
                            docker.image("${IMAGE_NAME}:${VERSION}").push('latest')
                        }
                        echo "Image pushed to Docker Hub successfully"
                    } catch (e) {
                        echo "Failed to push image: ${e}"
                        currentBuild.result = 'FAILURE'
                        error('Failed to push image')
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed - cleaning up'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
