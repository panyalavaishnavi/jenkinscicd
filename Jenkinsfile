
pipeline {
    agent none

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        APP_NAME = 'jenkins-cicd-app'
        REGISTRY_PUSH = 'localhost:5000'
        REGISTRY_PULL = 'host.minikube.internal:5000'
        DOCKER_PATH = 'C:\\Users\\user\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin'
        KUBECONFIG = 'C:\\Users\\user\\.kube\\config'
        MINIKUBE_HOME = 'C:\\Users\\user'
    }

    stages {
        stage('Checkout and Build') {
            agent { label 'staging-agent' }
            steps {
                checkout scm
                bat '''
                    set PATH=%DOCKER_PATH%;C:\\Program Files\\Kubernetes\\Minikube;%PATH%
                    echo ===== Python syntax check =====
                    python -m py_compile app.py
                    echo ===== Build version %BUILD_NUMBER% =====
                    docker build -t %REGISTRY_PUSH%/%APP_NAME%:%BUILD_NUMBER% .
                '''
            }
        }

        stage('Trivy Security Scan') {
            agent { label 'staging-agent' }
            steps {
                bat '''
                    set PATH=%DOCKER_PATH%;%PATH%
                    echo ===== Scanning image =====
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.75.0 image --exit-code 1 --severity CRITICAL %REGISTRY_PUSH%/%APP_NAME%:%BUILD_NUMBER%
                '''
            }
        }

        stage('Push Image') {
            agent { label 'staging-agent' }
            steps {
                bat '''
                    set PATH=%DOCKER_PATH%;%PATH%
                    docker push %REGISTRY_PUSH%/%APP_NAME%:%BUILD_NUMBER%
                '''
            }
        }

        stage('Deploy to Staging') {
            agent { label 'staging-agent' }
            steps {
                bat '''
                    set PATH=%DOCKER_PATH%;C:\\Program Files\\Kubernetes\\Minikube;%PATH%
                    set KUBECONFIG=%KUBECONFIG%
                    set MINIKUBE_HOME=%MINIKUBE_HOME%

                    kubectl create deployment %APP_NAME% --image=%REGISTRY_PULL%/%APP_NAME%:%BUILD_NUMBER% -n staging --dry-run=client -o yaml | kubectl apply -f -
                    if errorlevel 1 exit /b 1

                    kubectl rollout status deployment/%APP_NAME% -n staging --timeout=180s
                    if errorlevel 1 exit /b 1

                    kubectl expose deployment %APP_NAME% --name=%APP_NAME% --port=8080 --target-port=8080 --type=NodePort -n staging --dry-run=client -o yaml | kubectl apply -f -
                    if errorlevel 1 exit /b 1

                    kubectl exec deployment/%APP_NAME% -n staging -- python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:8080/health', timeout=5).read().decode())"
                '''
            }
        }

        stage('Approve Production') {
            steps {
                input message: 'Staging passed. Approve this image version for production?', ok: 'Approve production'
            }
        }

        stage('Deploy to Production') {
            agent { label 'production-agent' }
            steps {
                bat '''
                    set PATH=%DOCKER_PATH%;C:\\Program Files\\Kubernetes\\Minikube;%PATH%
                    set KUBECONFIG=%KUBECONFIG%
                    set MINIKUBE_HOME=%MINIKUBE_HOME%

                    kubectl create deployment %APP_NAME% --image=%REGISTRY_PULL%/%APP_NAME%:%BUILD_NUMBER% -n production --dry-run=client -o yaml | kubectl apply -f -
                    if errorlevel 1 exit /b 1

                    kubectl rollout status deployment/%APP_NAME% -n production --timeout=180s
                    if errorlevel 1 exit /b 1

                    kubectl expose deployment %APP_NAME% --name=%APP_NAME% --port=8080 --target-port=8080 --type=NodePort -n production --dry-run=client -o yaml | kubectl apply -f -
                    if errorlevel 1 exit /b 1

                    kubectl exec deployment/%APP_NAME% -n production -- python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:8080/health', timeout=5).read().decode())"
                '''
            }
            post {
                failure {
                    echo 'Production deployment failed. Attempting rollback to the previous revision.'
                    bat '''
                        set PATH=%DOCKER_PATH%;C:\\Program Files\\Kubernetes\\Minikube;%PATH%
                        set KUBECONFIG=%KUBECONFIG%
                        kubectl rollout undo deployment/%APP_NAME% -n production
                        if errorlevel 1 echo No previous deployment revision could be restored. Check the production deployment manually.
                    '''
                }
            }
        }
    }
}
