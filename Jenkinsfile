pipeline {
    agent any

    environment {
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
                use the below incase you have username and password of the registry
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'}
                echo 'Login successfully'
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
                    withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                    sed -i 's|{{IMAGE}}|${IMAGE_TAG}|g' deployment.yaml
                    kubectl apply -f deployment.yaml
                    '''
                    }
                }
            }
        }       
    }
}
