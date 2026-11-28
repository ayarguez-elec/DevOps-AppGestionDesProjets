pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_USER = 'ayarguez'
        BACKEND_IMAGE  = "${DOCKERHUB_USER}/backend-app:1.0"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/frontend-app:1.0"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Unit Tests') {
            steps {
                dir('backend') {
                    sh 'mvn test'
                }
            }
        }
        stage('Build Deliverable') {
            steps {
                dir('backend') {
                    sh 'mvn package -DskipTests'
                    sh 'ls -lh target/*.jar'
                }
            }
        }
        stage('Build Docker Images') {
            steps {
                dir('backend') {
                    sh "docker build -t ${BACKEND_IMAGE} ."
                }
                dir('frontend') {
                    sh "docker build -t ${FRONTEND_IMAGE} ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${BACKEND_IMAGE}"
                    sh "docker push ${FRONTEND_IMAGE}"
                }
            }
        }
        stage('Deploy with Docker Compose') {
            steps {
                sh 'docker compose down || true'
                sh 'docker compose up -d'
                sh 'docker ps'
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        failure {
            mail to: 'aya.rguez@esprit.tn',
                 subject: "Echec du build Jenkins : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "La pipeline a echoue lors du build #${env.BUILD_NUMBER}.\nConsulte les logs ici : ${env.BUILD_URL}"
        }
    }
}
