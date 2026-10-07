pipeline {
    agent any

    stages {

        stage('Check Jenkins Environment') {
            steps {
                bat '''
                echo ===== Jenkins Environment =====
                whoami
                echo USERPROFILE=%USERPROFILE%
                echo BUILD_NUMBER=%BUILD_NUMBER%
                echo KUBECONFIG=%KUBECONFIG%
                echo MINIKUBE_HOME=%MINIKUBE_HOME%
                where kubectl
                where minikube
                kubectl version --client
                minikube version
                '''
            }
        }

        stage('Build') {
            steps {
                bat '''
                set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH%
                echo ===== Building Version %BUILD_NUMBER% =====
                docker build -t jenkins-cicd-app:%BUILD_NUMBER% .
                '''
            }
        }

        stage('Security Scan') {
            steps {
                bat '''
                set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH%
                echo ===== Trivy Security Scan =====
                docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.75.0 image jenkins-cicd-app:%BUILD_NUMBER%
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                bat '''
                set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH%
                set MINIKUBE_HOME=C:\\Users\\user
                set KUBECONFIG=C:\\Users\\user\\.kube\\config

                echo ===== Minikube Profile =====
                minikube profile list

                echo ===== Loading Image Version %BUILD_NUMBER% =====
                minikube image load jenkins-cicd-app:%BUILD_NUMBER%

                echo ===== Updating Kubernetes Deployment =====
                kubectl set image deployment/jenkins-cicd-app jenkins-cicd-app=jenkins-cicd-app:%BUILD_NUMBER% -n cicd

                echo ===== Checking Deployment =====
                kubectl rollout status deployment/jenkins-cicd-app -n cicd
                '''
            }
        }

        stage('Approval') {
            steps {
                input message: 'Approve production deployment?', ok: 'Deploy'
            }
        }

        stage('Rollback') {
            steps {
                input message: 'Trigger Kubernetes rollback?', ok: 'Rollback'
                bat '''
                set PATH=C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;%PATH%
                set MINIKUBE_HOME=C:\\Users\\user
                set KUBECONFIG=C:\\Users\\user\\.kube\\config

                echo ===== Rolling Back Deployment =====
                kubectl rollout undo deployment/jenkins-cicd-app -n cicd

                echo ===== Verifying Rollback =====
                kubectl rollout status deployment/jenkins-cicd-app -n cicd
                '''
            }
        }
    }
}