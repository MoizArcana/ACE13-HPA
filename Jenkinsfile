pipeline {
    agent any

    environment {
        # Jenkins user kubeconfig
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        # Add snap kubectl path so Jenkins can find it
        PATH = "/snap/bin:${env.PATH}"
    }

    stages {

        stage('Workspace Debug') {
            steps {
                echo "Check current workspace and files:"
                sh 'pwd'
                sh 'ls -al'
            }
        }

        stage('Environment Debug') {
            steps {
                echo "Running as user:"
                sh 'whoami'
                
                echo "PATH environment:"
                sh 'echo $PATH'
                
                echo "Check kubectl version:"
                sh 'which kubectl'
                sh 'kubectl version --client'
                
                echo "Check nodes in cluster:"
                sh '/snap/bin/kubectl get nodes -o wide'
                
                echo "Kubeconfig path:"
                sh 'ls -l $KUBECONFIG'
            }
        }

        stage('Create Namespace') {
            steps {
                echo "Creating PROD namespace..."
                sh '/snap/bin/kubectl apply -f namespace.yaml'
                sh '/snap/bin/kubectl get ns'
            }
        }

        stage('Deploy ACE') {
            steps {
                echo "Deploying ACE pod and HPA..."
                sh '/snap/bin/kubectl apply -f deployment.yaml'
                sh '/snap/bin/kubectl apply -f hpa.yaml'

                echo "Verify deployment and pods:"
                sh '/snap/bin/kubectl get all -n PROD'
                sh '/snap/bin/kubectl describe deployment ace-deployment -n PROD'

                echo "Verify HPA status:"
                sh '/snap/bin/kubectl get hpa -n PROD'
                sh '/snap/bin/kubectl describe hpa ace-hpa -n PROD'
            }
        }

        stage('Create NodePort Service') {
            steps {
                echo "Exposing ACE pod via NodePort..."
                sh '''
                cat <<EOF | /snap/bin/kubectl apply -f -
                apiVersion: v1
                kind: Service
                metadata:
                  name: ace-nodeport
                  namespace: PROD
                spec:
                  type: NodePort
                  selector:
                    app: ace
                  ports:
                    - protocol: TCP
                      port: 7600
                      targetPort: 7600
                      nodePort: 30060
                EOF
                '''
                echo "NodePort service created. Access ACE at: http://<minikube_ip>:30060"
            }
        }

        stage('Verify ACE Pod') {
            steps {
                echo "Check pod logs (first 50 lines) for ACE container:"
                sh '''
                POD=$(/snap/bin/kubectl get pods -n PROD -l app=ace -o jsonpath="{.items[0].metadata.name}")
                /snap/bin/kubectl logs -n PROD $POD --tail=50
                '''
            }
        }

    }

    post {
        success {
            echo '✅ ACE deployed successfully with HPA and NodePort service'
        }
        failure {
            echo '❌ Deployment failed, check console output for errors'
        }
    }
}

