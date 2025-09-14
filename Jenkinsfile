pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '287043460198'
        AWS_REGION = 'ap-southeast-1'
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/app1"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        NAMESPACE = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/sabarees48/app1.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                    # Apply YAMLs (creates resources if not exist)
                    kubectl apply -f deployment.yaml -n ${NAMESPACE}
                    kubectl apply -f service.yaml -n ${NAMESPACE}
                    kubectl apply -f ingress.yaml -n ${NAMESPACE}

                    # Update Deployment with new image
                    kubectl set image deployment/app1 app1=${ECR_REPO}:${IMAGE_TAG} -n ${NAMESPACE}
                    kubectl rollout status deployment/app1 -n ${NAMESPACE}
                """
            }
        }
    }
}
