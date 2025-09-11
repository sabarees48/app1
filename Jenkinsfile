pipeline {
    agent any

    environment {
        AWS_REGION = "ap-southeast-1"       // 🔹 Change to your region
        ECR_REPO = "287043460198.dkr.ecr.ap-southeast-1.amazonaws.com/nodeapp1" // 🔹 Your ECR repo
        CLUSTER_NAME = "server-deployment"  // 🔹 Your EKS cluster name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sabarees48/app1.git'
            }
        }

        stage('Set IMAGE_TAG') {
            steps {
                script {
                    IMAGE_TAG = sh(script: "date +%Y%m%d%H%M%S", returnStdout: true).trim()
                }
            }
        }

        stage('Build & Push Image') {
            steps {
                script {
                    sh """
                    docker build -t $ECR_REPO:$IMAGE_TAG .
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    docker push $ECR_REPO:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                    sed -i 's|<ECR_URI>|$ECR_REPO|g' deployment.yaml
                    sed -i 's|:latest|:$IMAGE_TAG|g' deployment.yaml
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    """
                }
            }
        }

        stage('Install AWS Load Balancer Controller') {
            steps {
                script {
                    def controllerExists = sh(
                        script: "kubectl get deployment -n kube-system aws-load-balancer-controller --ignore-not-found",
                        returnStatus: true
                    )
                    if (controllerExists != 0) {
                        echo "Installing AWS Load Balancer Controller..."
                        sh """
                        helm repo add eks https://aws.github.io/eks-charts || true
                        helm repo update

                        helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
                          -n kube-system \
                          --set clusterName=$CLUSTER_NAME \
                          --set serviceAccount.create=false \
                          --set serviceAccount.name=aws-load-balancer-controller
                        """
                    } else {
                        echo "AWS Load Balancer Controller already installed ✅"
                    }
                }
            }
        }
    }
}
