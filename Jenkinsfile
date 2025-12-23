pipeline {
    agent any

    environment {
        # Jenkins user kubeconfig
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Kubectl Debug') {
            steps {
                echo "Running kubectl as user:"
                sh 'whoami'
                
                echo "Kubectl client version:"
                sh 'kubectl version --client'
                
                echo "Check nodes in cluster:"
                sh 'kubectl get nodes -o wide'
                
                echo "Check kubeconfig path:"
                sh 'echo $KUBECONFIG'
                sh 'ls -l $KUBECONFIG'
            }
        }

        stage('Checkout Git') {
            steps {
                git branch: 'main', url: 'https://github.com/MoizArcana/ACE13-HPA.git'
            }
        }

        stage('Create Namespace') {
            steps {
                echo "Creating PROD namespace..."
                sh 'kubectl apply -f namespace.yaml'
                sh 'kubectl get ns'
            }
        }

        stage('Deploy ACE') {
            steps {
                echo "Deploying ACE pod and HPA..."
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f hpa.yaml'
                
                echo "Verify deployment and pods:"
                sh 'kubectl get all -n PROD'
                sh 'kubectl describe deployment ace-deployment -n PROD'
                
                echo "Verify HPA status:"
                sh 'kubectl get hpa -n PROD'
                sh 'kubectl describe hpa ace-hpa -n PROD'
            }
        }

        stage('Verify ACE Pod') {
            steps {
                echo "Check pod logs (first 50 lines) for ace container:"
                sh 'kubectl logs -n PROD $(kubectl get pods -n PROD -l app=ace -o jsonpath="{.items[0].metadata.name}") --tail=50'
            }
        }
    }

    post {
        success {
            echo '✅ ACE deployed successfully with HPA enabled'
        }
        failure {
            echo '❌ Deployment failed, check console output for errors'
        }
    }
}

