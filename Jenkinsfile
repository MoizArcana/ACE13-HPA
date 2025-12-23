pipeline {
    agent any

    environment {
        KUBECONFIG = "${env.HOME}/.kube/config"
    }

    stages {

        stage('Kubectl Debug') {
            steps {
                sh 'whoami'
                sh 'kubectl version --client'
                sh 'kubectl get nodes'
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'YOUR_GITHUB_REPO_URL'
            }
        }

        stage('Create Namespace') {
            steps {
                sh 'kubectl apply -f namespace.yaml'
            }
        }

        stage('Deploy ACE') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f hpa.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get all -n PROD
                kubectl get hpa -n PROD
                '''
            }
        }

    } // end of stages

    post {
        success {
            echo '✅ ACE deployed successfully with HPA enabled'
        }
        failure {
            echo '❌ Deployment failed'
        }
    }
}
