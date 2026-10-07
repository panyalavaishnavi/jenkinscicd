pipeline {
    agent any

    stages {

        stage('Check Jenkins Environment') {
            steps {
                bat 'whoami'
                bat 'echo USERPROFILE=%USERPROFILE%'
                bat 'set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH% && set KUBECONFIG=C:\\Users\\user\\.kube\\config && kubectl config current-context'
                bat 'set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH% && set KUBECONFIG=C:\\Users\\user\\.kube\\config && kubectl get nodes'
            }
        }

        stage('Build') {
            steps {
                bat 'set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH% && docker build -t jenkins-cicd-app:v1 .'
            }
        }

        stage('Security Scan') {
            steps {
                bat 'set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH% && docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.75.0 image jenkins-cicd-app:v1'
            }
        }

        stage('Deploy to Minikube') {
            steps {
                bat 'set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH% && set KUBECONFIG=C:\\Users\\user\\.kube\\config && minikube image load jenkins-cicd-app:v1'
                bat 'set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH% && set KUBECONFIG=C:\\Users\\user\\.kube\\config && kubectl set image deployment/jenkins-cicd-app jenkins-cicd-app=jenkins-cicd-app:v1 -n cicd'
            }
        }

        stage('Approval') {
            steps {
                input message: 'Approve production deployment?', ok: 'Deploy'
            }
        }
    }
}