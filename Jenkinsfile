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
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'jenkins-config-v2', url: 'https://github.com/saradrine/application-one.git'
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


        stage('Tests (optionnel)') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:$VERSION ."
            }
        }

        stage('Push vers Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
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
        success {
            echo 'Déploiement terminé avec succès.'
        }
        failure {
            echo 'Échec du pipeline.'
        }
    }
}
