pipeline {
    agent any

    stages {
       stage('Build') {
    steps {
        bat 'set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH% && docker build -t jenkins-cicd-app:v1 .'
    }
}
        stage('Security Scan') {
            steps {
                bat 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.75.0 image jenkins-cicd-app:v1'
            }
        }

        stage('Deploy to Minikube') {
            steps {
                bat 'minikube image load jenkins-cicd-app:v1'
                bat 'kubectl set image deployment/jenkins-cicd-app jenkins-cicd-app=jenkins-cicd-app:v1 -n cicd'
            }
        }

        stage('Approval') {
            steps {
                input message: 'Approve production deployment?', ok: 'Deploy'
            }
        }
    }
}