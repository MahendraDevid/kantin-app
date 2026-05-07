// Jenkinsfile
pipeline {
    agent any

    environment {
        // Ganti dengan username Docker Hub kamu
        DOCKER_USER = "USERNAME_DOCKER_HUB_KAMU"
        // Ganti dengan URL repo GitHub kamu
        GIT_REPO_URL = "https://github.com/USERNAME_GITHUB/kantin-app.git"
    }

    stages {

        // TAHAP 1: Ambil kode terbaru dari GitHub
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO_URL}"
            }
        }

        // TAHAP 2: Build Docker image + Push ke Docker Hub
        stage('Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-login',
                    passwordVariable: 'PASS',
                    usernameVariable: 'USER'
                )]) {
                    // Login ke Docker Hub
                    sh "echo ${PASS} | docker login -u ${USER} --password-stdin"

                    // Build image backend
                    sh "docker build -t ${USER}/kantin-backend:latest ./backend"

                    // Build image frontend
                    sh "docker build -t ${USER}/kantin-frontend:latest ./frontend"

                    // Push ke Docker Hub
                    sh "docker push ${USER}/kantin-backend:latest"
                    sh "docker push ${USER}/kantin-frontend:latest"
                }
            }
        }

        // TAHAP 3: Deploy ke AKS
        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: 'aks-config', variable: 'KUBECONFIG')]) {
                    // Apply Kubernetes manifests
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply -f kantin-k8s.yaml"
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply -f kantin-ingress.yaml"

                    // Paksa rolling restart agar image terbaru dipakai
                    sh "kubectl --kubeconfig=${KUBECONFIG} rollout restart deployment backend-kantin"
                    sh "kubectl --kubeconfig=${KUBECONFIG} rollout restart deployment frontend-kantin"

                    // Cek status deploy
                    sh "kubectl --kubeconfig=${KUBECONFIG} get pods"
                    sh "kubectl --kubeconfig=${KUBECONFIG} get ingress"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline berhasil! Aplikasi sudah ter-deploy ke AKS.'
        }
        failure {
            echo '❌ Pipeline gagal. Cek Console Output untuk detail error.'
        }
    }
}