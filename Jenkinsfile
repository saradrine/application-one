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
        JAVA_HOME = '/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                sh 'git --version'
                echo "Code checked out from ${env.GIT_URL}"
            }
        }

        stage('Verify Java Version') {
            steps {
                sh '''
                    echo "JAVA_HOME: ${JAVA_HOME}"
                    java -version
                    mvn -v
                '''
            }
        }

        stage('Build with Maven') {
            steps {
                script {
                    try {
                        sh '''
                            mvn -v
                            mvn clean package -DskipTests
                        '''
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
                        sh '/usr/local/bin/docker --version'
                        sh "/usr/local/bin/docker build -t ${IMAGE_NAME}:${VERSION}-${BUILD_DATE} ."
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
                        sh "/usr/local/bin/docker login -u ${DOCKER_HUB_USR} -p ${DOCKER_HUB_PSW} https://registry.hub.docker.com"
                        sh "/usr/local/bin/docker push ${IMAGE_NAME}:${VERSION}-${BUILD_DATE}"
                        sh "/usr/local/bin/docker tag ${IMAGE_NAME}:${VERSION}-${BUILD_DATE} ${IMAGE_NAME}:latest"
                        sh "/usr/local/bin/docker push ${IMAGE_NAME}:latest"
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