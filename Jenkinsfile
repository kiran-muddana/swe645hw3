pipeline {
    agent any
    tools {
        jdk 'java'
        maven 'maven'
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
                sh 'docker build -t skm05/survey:latest .'
            }
        }
        stage('Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push image to Docker Hub') {
            steps {
                sh 'docker push skm05/survey:latest'
            }
        }
        stage('Deploying on Kubernetes') {
            steps {
                // Set the image for the Kubernetes deployment
                sh "kubectl set image deployment/back container-0=${IMAGE_TAG} -n default"
                // Restart the deployment to apply changes
                sh "kubectl rollout restart deploy back -n default"
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
