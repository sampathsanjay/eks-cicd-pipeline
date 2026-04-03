pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'sampathsanjay/flask-app'
        IMAGE_TAG      = "v${BUILD_NUMBER}"
        CLUSTER_NAME   = 'my-eks-cluster'
        AWS_REGION     = 'us-east-1'
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/sampathsanjay/eks-cicd-pipeline.git',
                    branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                        docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest
                        docker push ${DOCKERHUB_REPO}:latest
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
                    sed -i 's|IMAGE_TAG|${DOCKERHUB_REPO}:${IMAGE_TAG}|g' k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/flask-app --timeout=120s
                """
            }
        }

    }

    post {
        success {
            echo "✅ Deployment successful - ${DOCKERHUB_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed - check logs"
        }
    }
}
