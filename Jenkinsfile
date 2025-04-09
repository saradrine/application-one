pipeline {
    agent {
        docker {
            image 'node:22.14.0-alpine3.21'
            args '-v /home/jenkins/.m2:/root/.m2'  // Optional: Cache Maven dependencies
        }
    }

    tools {
            maven 'M3'
            jdk 'jdk17'
    }

    environment {
        DOCKER_IMAGE = 'rymjbeli/application-one'
        VERSION = "${new Date().format('yyyyMMdd-HHmm')}"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        dockerImage = ''
        registery = 'rymjbeli/application-one'
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'jenkins-config-v2', url: 'https://github.com/saradrine/application-one.git'
            }
        }

        // stage('Build with Maven') {
        //     steps {
        //          script {
        //             // This will automatically pull the image if needed
        //             docker.image('node:22.14.0-alpine3.21').inside {
        //                 sh 'node --version'
        //                 sh 'npm install'
        //                 sh 'npm run build'
        //             }
        //         }
        //     }

        //     post {
        //         success {
        //             echo 'Maven build completed successfully!'
        //             stash includes: 'target/*.jar', name: 'app-jar'
        //         }
        //     }
        // }

        stage('Build') {
            steps {
                sh 'node --version'
                sh 'npm install'
                sh 'npm run build'
            }
        }


        stage('Tests (optionnel)') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                // sh "docker build -t $DOCKER_IMAGE:$VERSION ."
                dockerImage = docker.build.registery
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
