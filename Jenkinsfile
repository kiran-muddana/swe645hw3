pipeline {
    agent any
    tools {
        jdk 'java'
        maven 'maven'
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_TAG = "skm05/survey:${BUILD_NUMBER}"
    }
    stages {
        stage('Maven Clean') {
            steps {
               script{
                sh 'mvn clean install -DskipTests'
               }
            }
        }
        
        stage('Build') {
            steps {
                sh 'rm -rf *.var'
                sh 'jar -cvf survey0.1-0.0.1-SNAPSHOT.jar -C "src/main" .'     
                sh 'docker build -t skm05/survey:latest .'
            }
        }
        stage('Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage("Push image to docker hub") {
            steps {
                sh 'docker push skm05/survey:latest'
            }
        }
        stage("deploying on k8") {
            steps {
                sh 'kubectl set image deployment/back container-0=skm05/survey:latest -n default'
                sh 'kubectl rollout restart deploy back -n default'
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
