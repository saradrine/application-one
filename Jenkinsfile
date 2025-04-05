pipeline {
    agent any

    environment {
        // Configuration Docker Hub
        DOCKER_HUB = credentials('docker-hub-credentials')
        IMAGE_NAME = 'sara12308/application-one'
        VERSION = "${env.BUILD_NUMBER}"
        BUILD_DATE = new Date().format('yyyyMMdd-HHmmss')

        // Configuration Maven
        MAVEN_OPTS = "-Dmaven.repo.local=${env.WORKSPACE}/.m2/repository"
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

                        // Construit l'image
                        dockerImage = docker.build("${IMAGE_NAME}:${VERSION}-${BUILD_DATE}")

                        echo "Image ${IMAGE_NAME}:${VERSION} built successfully"
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
                            dockerImage.push()
                            dockerImage.push('latest')
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
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            slackSend(color: 'good', message: "Pipeline SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            echo 'Pipeline failed!'
            slackSend(color: 'danger', message: "Pipeline FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}