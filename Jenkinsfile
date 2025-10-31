@Library('testsharedlibry') _

pipeline {
    agent any

    parameters {
       string(name: 'FRONTEND', defaultValue: 'frontend', description: 'this is my docker image name for frontend')
       string(name: 'ORDERSERVICE', defaultValue: 'orderservice', description: 'this is my docker image for order service')
       string(name: 'PRODUCTSERVICE', defaultValue: 'productservice', description: 'this is my docker image for product service')
       string(name: 'USERSERVICE', defaultValue: 'userservice', description: 'this is my docker image for user service')
       string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'this is my docker tag name')
       string(name: 'CONTAINER_FRONTEND', defaultValue: 'frontend_container', description: 'this is my docker container name for frontend')
       string(name: 'CONTAINER_ORDERSERVICE', defaultValue: 'orderservice_container', description: 'this is my docker container name for orderservice')
       string(name: 'CONTAINER_PRODUCTSERVICE', defaultValue: 'productservice_container', description: 'this is my docker container name for productservice')
       string(name: 'CONTAINER_USERSERVICE', defaultValue: 'user_container', description: 'this is my docker container name for userservice')
       string(name: 'GITHUB_USER', defaultValue: 'DonovanKen', description: 'user')
       string(name: 'DOCKERHUB_USER', defaultValue: 'mrkendono', description: 'docker hub')
       string(name: 'K8S_NAMESPACE', defaultValue: 'default', description: 'Kubernetes namespace')
    }
    
    stages {
      stage('Build Image') {
        steps {
          script {
           dockerBuild("$FRONTEND", "$ORDERSERVICE", "$PRODUCTSERVICE", "$USERSERVICE", "$IMAGE_TAG")
          }
        }
      }
      
      stage('Push Images to Docker Hub') {
        steps {
            script {
                sh """
                    docker tag ${params.FRONTEND}:${params.IMAGE_TAG} ${params.DOCKERHUB_USER}/${params.FRONTEND}:${params.IMAGE_TAG}
                    docker push ${params.DOCKERHUB_USER}/${params.FRONTEND}:${params.IMAGE_TAG}
                """
                sh """
                    docker tag ${params.ORDERSERVICE}:${params.IMAGE_TAG} ${params.DOCKERHUB_USER}/${params.ORDERSERVICE}:${params.IMAGE_TAG}
                    docker push ${params.DOCKERHUB_USER}/${params.ORDERSERVICE}:${params.IMAGE_TAG}
                """
                sh """
                    docker tag ${params.PRODUCTSERVICE}:${params.IMAGE_TAG} ${params.DOCKERHUB_USER}/${params.PRODUCTSERVICE}:${params.IMAGE_TAG}
                    docker push ${params.DOCKERHUB_USER}/${params.PRODUCTSERVICE}:${params.IMAGE_TAG}
                """
                sh """
                    docker tag ${params.USERSERVICE}:${params.IMAGE_TAG} ${params.DOCKERHUB_USER}/${params.USERSERVICE}:${params.IMAGE_TAG}
                    docker push ${params.DOCKERHUB_USER}/${params.USERSERVICE}:${params.IMAGE_TAG}
                """
            }
        }
      }

      stage('Run Container Test') {
        steps {
            script {
                testimage("$FRONTEND", "$ORDERSERVICE", "$PRODUCTSERVICE", "$USERSERVICE", "$IMAGE_TAG", "$CONTAINER_FRONTEND", "$CONTAINER_ORDERSERVICE", "$CONTAINER_PRODUCTSERVICE", "$CONTAINER_USERSERVICE")
            }
        }
      }

      stage('Deploy to Kubernetes') {
    steps {
        sshagent(credentials: ['ssh-cred1']) {
            sh '''
                [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                ssh-keyscan -t rsa,dsa 192.168.2.88 >> ~/.ssh/known_hosts
            '''
            
            sh """
                # Create directory on remote server first
                ssh -o StrictHostKeyChecking=no kubernetes@192.168.2.88 'mkdir -p /tmp/k8s-manifests/'
                
                # Copy k8s manifests to target server
                scp -r k8s/ kubernetes@192.168.2.88:/tmp/k8s-manifests/
                
                # Deploy to Kubernetes - FIXED PATH
                ssh kubernetes@192.168.2.88 "
                    cd /tmp/k8s-manifests/k8s
                    echo '=== Current directory and files ==='
                    pwd
                    ls -la
                    echo '=== Applying Kubernetes manifests ==='
                    kubectl apply -f .
                    echo '=== Waiting for deployments to be ready ==='
                    kubectl wait --for=condition=available deployment/frontend-deployment --timeout=300s
                    kubectl wait --for=condition=available deployment/userservice-deployment --timeout=300s
                    kubectl wait --for=condition=available deployment/productservice-deployment --timeout=300s
                    kubectl wait --for=condition=available deployment/orderservice-deployment --timeout=300s
                    echo '=== Deployment Status ==='
                    kubectl get deployments
                    echo '=== Pod Status ==='
                    kubectl get pods
                    echo '=== Service Status ==='
                    kubectl get services
                "
            """
        }
    }
}
    }
    
    post {
        always {
            echo 'Pipeline completed - check Kubernetes deployment status'
        }
        success {
            echo 'All microservices deployed successfully to Kubernetes!'
        }
        failure {
            echo 'Deployment failed - check Kubernetes logs'
        }
    }
}