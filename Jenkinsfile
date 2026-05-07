// Jenkinsfile
pipeline {
    agent any

    environment {
        // Ganti dengan username Docker Hub kamu
        DOCKER_USER = "madeu30"
        // Ganti dengan URL repo GitHub kamu
        GIT_REPO_URL = "https://github.com/MahendraDevid/kantin-app.git"
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
                    sh 'echo $PASS | docker login -u $USER --password-stdin'

                    // Nama image harus sama dengan yang di kantin-k8s.yaml
                    sh "docker build -t $USER/praktikum1-backend-service:latest ./backend"
                    sh "docker build -t $USER/praktikum1-frontend-service:latest ./frontend"
                    sh "docker push $USER/praktikum1-backend-service:latest"
                    sh "docker push $USER/praktikum1-frontend-service:latest"
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