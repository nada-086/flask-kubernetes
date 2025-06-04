pipeline {
    agent any

    environment {
        // KUBECONFIG = credentials('kubeconfig-credentials-id')
        // AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        // AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        AZURE_SUBSCRIPTION_ID = credentials('AZURE_SUBSCRIPTION_ID')
        AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
        AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
        AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
        ACR_NAME = 'your-acr-name'
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        AKS_RESOURCE_GROUP = 'your-aks-resource-group'
        AKS_CLUSTER_NAME = 'your-aks-cluster-name'
        IMAGE_NAME = "${ACR_LOGIN_SERVER}/jenkins/jenkins-flask-app"
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
        
    }

    
    stages {
        stage('Setup') {
            steps {
                // use the below if you are using kube-config file
                // sh 'ls -la $KUBECONFIG'
                // sh 'chmod 644 $KUBECONFIG'
                // sh 'ls -la $KUBECONFIG'
                sh "pip install -r requirements.txt"
            }
        }
        stage('Test') {
            steps {
                sh "pytest"
            }
        }

        stage('Login to docker hub') {
            steps {
                // use the below incase you have username and password of the registry
                // withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                // sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'}
                // echo 'Login successfully'

                // otherwise I am already connected to my acr with azure credentials
                // Authenticate with Azure
                sh '''
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
                    az account set -s $AZURE_SUBSCRIPTION_ID
                    '''
                // Authenticate Docker with ACR
                sh "az acr login --name ${ACR_NAME}"
                }

            }
        }
        stage('Build Docker Image')
        {
            steps
            {
                sh 'docker build -t ${IMAGE_TAG} .'
                echo "Docker image build successfully"
                sh 'docker image ls'
                
            }
        }

        stage('Push Docker Image')
        {
            steps
            {
                sh 'docker push ${IMAGE_TAG}'
                echo "Docker image push successfully"
            }
        }

        stage('Deploy to kubernetes')
        {
            steps {
                script {
                    // Get AKS credentials
                    sh "az aks get-credentials --resource-group ${AKS_RESOURCE_GROUP} --name ${AKS_CLUSTER_NAME}"

                    // Apply Kubernetes manifests (assuming you have a deployment.yaml)
                    sh '''
                    sed -i 's|{{IMAGE}}|${IMAGE_TAG}|g' deployment.yaml
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }       
}
