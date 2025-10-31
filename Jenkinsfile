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
                // Tag and push frontend
                sh """
                    docker tag ${params.FRONTEND}:${params.IMAGE_TAG} ${params.DOCKERHUB_USER}/${params.FRONTEND}:${params.IMAGE_TAG}
                    docker push ${params.DOCKERHUB_USER}/${params.FRONTEND}:${params.IMAGE_TAG}
                """
                
                // Tag and push orderservice
                sh """
                    docker tag ${params.ORDERSERVICE}:${params.IMAGE_TAG} ${params.DOCKERHUB_USER}/${params.ORDERSERVICE}:${params.IMAGE_TAG}
                    docker push ${params.DOCKERHUB_USER}/${params.ORDERSERVICE}:${params.IMAGE_TAG}
                """

                // Tag and push productservice
                sh """
                    docker tag ${params.PRODUCTSERVICE}:${params.IMAGE_TAG} ${params.DOCKERHUB_USER}/${params.PRODUCTSERVICE}:${params.IMAGE_TAG}
                    docker push ${params.DOCKERHUB_USER}/${params.PRODUCTSERVICE}:${params.IMAGE_TAG}
                """

                // Tag and push userservice
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
                
                // Copy k8s manifests to target server and deploy
                sh """
                    # Copy k8s directory to target server
                    scp -r k8s/ kubernetes@192.168.2.88:/tmp/k8s-manifests/
                    
                    # Deploy to Kubernetes
                    ssh kubernetes@192.168.2.88 "
                        cd /tmp/k8s-manifests
                        
                        # Apply all Kubernetes manifests
                        kubectl apply -f frontend-deployment.yaml
                        kubectl apply -f userservice-deployment.yaml  
                        kubectl apply -f productservice-deployment.yaml
                        kubectl apply -f orderservice-deployment.yaml
                        
                        # Wait for deployments to be ready
                        kubectl wait --for=condition=available deployment/frontend-deployment --timeout=300s
                        kubectl wait --for=condition=available deployment/userservice-deployment --timeout=300s
                        kubectl wait --for=condition=available deployment/productservice-deployment --timeout=300s
                        kubectl wait --for=condition=available deployment/orderservice-deployment --timeout=300s
                        
                        # Show deployment status
                        echo '=== Deployment Status ==='
                        kubectl get deployments
                        
                        echo '=== Service Status ==='
                        kubectl get services
                        
                        echo '=== Pod Status ==='
                        kubectl get pods
                    "
                """
            }
        }
      }
      
      stage('Verify Deployment') {
        steps {
            sshagent(credentials: ['ssh-cred1']) {
                sh """
                    ssh kubernetes@192.168.2.88 "
                        echo '=== Final Deployment Status ==='
                        kubectl get all
                        
                        echo '=== Pod Details ==='
                        kubectl get pods -o wide
                        
                        echo '=== Service URLs ==='
                        kubectl get services
                        
                        echo '=== Checking Pod Logs ==='
                        kubectl logs -l app=frontend --tail=10
                        kubectl logs -l app=userservice --tail=10
                        kubectl logs -l app=productservice --tail=10
                        kubectl logs -l app=orderservice --tail=10
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