pipeline {
    agent any

    environment {
        IMAGE_NAME = "vijay14082003/vijaychowdarykube"
        IMAGE_TAG = "v1"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl delete deployment demo --ignore-not-found=true
                kubectl create deployment demo --image=${IMAGE_NAME}:${IMAGE_TAG}
                kubectl expose deployment demo --type=NodePort --port=8080 --target-port=8080 --dry-run=client -o yaml | kubectl apply -f -
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'kubectl get all'
            }
        }
    }

    post {
        success {
            echo 'Pipeline Executed Successfully!'
        }

        failure {
            echo 'Pipeline Failed!'
        }
    }
}
