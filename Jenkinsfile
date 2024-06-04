pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        KUBECONFIG = credentials('kubeconfig')
        GITHUB_CREDENTIALS = credentials('github-credentials')
        ARGOCD_TOKEN = credentials('argocd-token')
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-credentials', url: 'https://github.com/mooazsayyed/devops_Assignment.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('railsapp:latest')
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        docker.image('railsapp:latest').push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(credentialsId: 'kubeconfig') {
                    sh 'kubectl apply -f kubernetes-manifests/'
                }
            }
        }

        stage('Deploy with ArgoCD') {
            steps {
                script {
                    sh """
                    argocd login --insecure --grpc-web --username admin --password ${ARGOCD_TOKEN}
                    argocd app sync railsapp
                    """
                }
            }
        }
    }
}
