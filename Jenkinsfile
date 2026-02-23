pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        EKS_CLUSTER_NAME = "docker_eks_test_cluster"

        // DockerHub credentials stored in Jenkins
        DOCKERHUB_CREDS = credentials('docker-cred')

        IMAGE_NAME = "${DOCKERHUB_CREDS_USR}/myapp"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/subratgithub/project-5.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t $FULL_IMAGE .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh '''
                    echo "$DOCKERHUB_CREDS_PSW" | \
                    docker login -u "$DOCKERHUB_CREDS_USR" --password-stdin
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    echo "Pushing image..."
                    docker push $FULL_IMAGE
                '''
            }
        }

        stage('Deploy to EKS') {
    steps {
        sh '''
            echo "Deploying to EKS..."

            aws sts get-caller-identity

            aws eks update-kubeconfig \
                --region $AWS_REGION \
                --name $EKS_CLUSTER_NAME

            kubectl get nodes

            sed -i "s|image:.*|image: $FULL_IMAGE|g" K8s/deployment.yaml

            kubectl apply -f K8s/deployment.yaml
            kubectl apply -f K8s/service.yaml

            kubectl rollout status deployment/dockerhub-sample-app
        '''
            }
        }
    }

    post {
        success {
            echo "Deployment successful."
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
        always {
            sh 'docker logout'
        }
    }
}
