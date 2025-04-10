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
                        sh "docker build -t ${IMAGE_NAME}:${VERSION}-${BUILD_DATE} ."
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
                withCredentials([usernamePassword(credentialsId: "docker-hub-credentials", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag $DOCKER_IMAGE:$VERSION $DOCKER_IMAGE:latest
                        docker push $DOCKER_IMAGE:$VERSION
                        docker push $DOCKER_IMAGE:latest
                    """
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
