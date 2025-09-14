pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID = '287043460198'
        AWS_REGION = 'ap-southeast-1'
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/app1"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = '/home/jenkins/.kube/config'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sabarees48/app1.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }
        stage('Push to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl set image deployment/app1 app1=$ECR_REPO:$IMAGE_TAG
                kubectl rollout status deployment/app1
                '''
            }
        }
    }
}
