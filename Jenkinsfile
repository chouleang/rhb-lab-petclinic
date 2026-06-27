pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = "chouleang/rhb-lab-petclinic"
        DOCKER_TAG = "${BUILD_NUMBER}"
        EKS_CLUSTER = "rhb-eks"
        AWS_REGION = "ap-southeast-1"
        NAMESPACE = "petclinics"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "=== Checking out code from GitHub ==="
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "=== Building Docker image ==="
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "=== Pushing to Docker Hub ==="
                sh """
                    echo ${DOCKER_HUB_CREDENTIALS_PSW} | \
                    docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

stage('Deploy to EKS') {
    steps {
        echo "=== Deploying to EKS ==="
        sh """
            aws eks update-kubeconfig \
                --name ${EKS_CLUSTER} \
                --region ${AWS_REGION}
        """
        
        script {
            // Check if deployment exists
            def deployExists = sh(
                script: "kubectl get deployment api-gateway -n ${NAMESPACE} 2>/dev/null",
                returnStatus: true
            ) == 0

            if (deployExists) {
                echo "=== Deployment exists → updating image ==="
                sh """
                    kubectl set image deployment/api-gateway \
                        api-gateway=${DOCKER_IMAGE}:${DOCKER_TAG} \
                        -n ${NAMESPACE}
                    
                    kubectl rollout status deployment/api-gateway \
                        -n ${NAMESPACE}
                """
            } else {
                echo "=== Deployment not found → creating from manifest ==="
                sh """
                    # Replace image placeholder in manifest
                    sed -i 's|DOCKER_HUB_USERNAME/rhb-lab-petclinic:latest|${DOCKER_IMAGE}:${DOCKER_TAG}|g' \
                        k8s/deployment.yaml
                    
                    kubectl apply -f k8s/ -n ${NAMESPACE}
                    
                    kubectl rollout status deployment/api-gateway \
                        -n ${NAMESPACE}
                """
            }
        }
    }
}
    post {
        success {
            echo "=== Pipeline SUCCESS ==="
            echo "Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "Deployed to EKS namespace: ${NAMESPACE}"
        }
        failure {
            echo "=== Pipeline FAILED ==="
        }
        always {
            sh "docker logout"
        }
    }
}
