pipeline {
    agent any
    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }
    options {
        timeout(time: 60, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    environment {
        BACK  = 'bilelbrini_5ds8_gestionprojets-backend'
        FRONT = 'bilelbrini_5ds8_gestionprojets-frontend'
        TAG   = '1.0.0'
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build Backend') {
            steps {
                dir('backend') { sh 'mvn clean package -DskipTests' }
            }
        }
        stage('Test Backend') {
            steps {
                dir('backend') { sh 'mvn test' }
            }
        }
        stage('Build Images') {
            steps {
                sh 'docker build -t $BACK:$TAG backend'
                sh 'docker build -t $FRONT:$TAG frontend'
            }
        }
        stage('Push DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker tag $BACK:$TAG $DOCKER_USER/$BACK:$TAG'
                    sh 'docker tag $FRONT:$TAG $DOCKER_USER/$FRONT:$TAG'
                    sh 'docker push $DOCKER_USER/$BACK:$TAG'
                    sh 'docker push $DOCKER_USER/$FRONT:$TAG'
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }
    post {
        always  { sh 'docker logout || true' }
        success { echo 'Pipeline OK' }
        failure { echo 'Pipeline en échec' }
    }
}
