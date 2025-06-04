pipeline {
    agent any

    environment {
        IMAGE_NAME = "nadaessa/jenkins-flask-app"
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
    }

    stages {
        stage('Setup') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                . venv/bin/activate
                pytest
            '''
            }
        }


        stage('Login to Docker Registry') {
            steps {
                // Use the below in case you have username and password of the registry
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'
                }
                echo 'Login successful'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_TAG} .'
                echo "Docker image built successfully"
                sh 'docker image ls'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push ${IMAGE_TAG}'
                echo "Docker image pushed successfully"
            }
        }

        stage('Deploy to Kubernetes') {
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
