pipeline {
    agent any

    environment {
        DOCKER_USER = "surya8442"
        IMAGE_NAME = "instagram"
        IMAGE_TAG = "v1"
        CONTAINER_NAME = "insta-cont"
        DOCKER_CREDENTIALS_ID = "Docker_cred"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Surya8442/instagram.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Run Container (Local Test)') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d -p 5000:5000 --name ${CONTAINER_NAME} ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Approval') {
            steps {
                input "Approve deployment to Kubernetes?"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    export KUBECONFIG=/var/lib/jenkins/.kube/config

                    kubectl get nodes

                    kubectl apply -f insta.yml
                    kubectl apply -f auto.yml
                    kubectl apply -f take.yml
                    kubectl apply -f food.yml
                '''
            }
        }

        stage('Deploy Ingress') {
            steps {
                sh '''
                    export KUBECONFIG=/var/lib/jenkins/.kube/config

                    kubectl apply -f ingress.yml

                    echo "======================================"
                    echo "Ingress deployed successfully 🌐"
                    echo "======================================"

                    kubectl get ingress
                '''
            }
        }

        stage('Show App URL') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Application deployed successfully 🚀"
                    echo "Access it at: http://myapp.local"
                    echo "======================================"
                '''
            }
        }
    }
}
