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
                sh 'docker build -t skm05/springdemo:latest .'
            }
        }
        stage('Build HTML Image') {
            steps {
                // Build the Docker image for the HTML application
                sh 'docker build -t skm05/frontdemo:latest -f Dockerfile .'
            }
        }
        stage('Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage("Push image to docker hub") {
            steps {
                sh 'docker push skm05/springdemo:latest'
            }
        }
        stage("Push HTML Image to Docker Hub") {
            steps {
                sh 'docker push skm05/frontdemo:latest'
            }
        }
        stage("deploying on k8") {
            steps {
                sh 'kubectl set image deployment/backenddemo container-0=skm05/springdemo:latest -n default'
                sh 'kubectl rollout restart deploy backenddemo -n default'

                sh "kubectl set image deployment/frontdemo container-0=skm05/frontdemo:latest -n default"
                sh "kubectl rollout restart deploy frontdemo -n default"
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
