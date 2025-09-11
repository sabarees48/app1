pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '287043460198'
        AWS_REGION     = 'ap-southeast-1'
        ECR_REPO       = 'nodeapp1'   // unique per API
        ECR_URI        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {
        stage('Set IMAGE_TAG') {
            steps {
                script {
                    def ts = new Date().format("yyyyMMddHHmmss")
                    env.IMAGE_TAG = ts
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sabarees48/app1.git'
            }
        }

        stage('Build & Push Image') {
            steps {
                sh """
                  docker build -t ${ECR_URI}:${IMAGE_TAG} .
                  aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}
                  docker push ${ECR_URI}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                  sed -i "s|<ECR_URI>|${ECR_URI}|g" deployment.yaml
                  sed -i "s|:latest|:${IMAGE_TAG}|g" deployment.yaml
                  kubectl apply -f deployment.yaml
                  kubectl apply -f service.yaml
                """
            }
        }
    }
}
