@Library('testsharedlibry') _

pipeline {
    agent any

    parameters {
       string(name: 'FRONTEND', defaultValue: 'frontend', description: 'this is my docker image name for frontend')
       string(name: 'ORDERSERVICE', defaultValue: 'orderservice', description: 'this is my docker image for order service')
       string(name: 'PRODUCTSERVICE', defaultValue: 'productservice', description: 'this is my docker image for product service')
       string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'this is my docker tag name')
       string(name: 'CONTAINER_FRONTEND', defaultValue: 'frontend_container', description: 'this is my docker container name for frontend')
       string(name: 'CONTAINER_ORDERSERVICE', defaultValue: 'orderservice_container', description: 'this is my docker container name for orderservice')
       string(name: 'CONTAINER_PRODUCTSERVICE', defaultValue: 'productservice_container', description: 'this is my docker container name for productservice')
       string(name: 'GITHUB_USER', defaultValue: 'DonovanKen', description: 'user')
       string(name: 'DOCKERHUB_USER', defaultValue: 'mrkendono', description: 'docker hub')
    }
    
    stages {
      stage('Build Image') {
        steps {
          script {
           dockerBuild("$FRONTEND", "$ORDERSERVICE", "$PRODUCTSERVICE", "$IMAGE_TAG")
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
            }
        }
      }

      stage('Run Container Test') {
        steps {
            script {
                testimage("$FRONTEND", "$ORDERSERVICE", "$PRODUCTSERVICE", "$IMAGE_TAG", "$CONTAINER_FRONTEND", "$CONTAINER_ORDERSERVICE", "$CONTAINER_PRODUCTSERVICE")
            }
        }
      }

      stage('Deploy') {
        steps {
            sshagent(credentials: ['ssh-cred']) {
                sh '''
                    [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    ssh-keyscan -t rsa,dsa 192.168.2.70 >> ~/.ssh/known_hosts
                '''
                
                // Deploy frontend
                sh """
                    ssh ken3@192.168.2.70 "
                        docker pull ${params.DOCKERHUB_USER}/${params.FRONTEND}:${params.IMAGE_TAG} &&
                        docker rm -f ${params.CONTAINER_FRONTEND} || true &&
                        docker run -d -p 5000:5000 --name ${params.CONTAINER_FRONTEND} ${params.DOCKERHUB_USER}/${params.FRONTEND}:${params.IMAGE_TAG}
                    "
                """
                
                // Deploy orderservice
                sh """
                    ssh ken3@192.168.2.70 "
                        docker pull ${params.DOCKERHUB_USER}/${params.ORDERSERVICE}:${params.IMAGE_TAG} &&
                        docker rm -f ${params.CONTAINER_ORDERSERVICE} || true &&
                        docker run -d -p 5003:5003 --name ${params.CONTAINER_ORDERSERVICE} ${params.DOCKERHUB_USER}/${params.ORDERSERVICE}:${params.IMAGE_TAG}
                    "
                """


                // Deploy productservice
                sh """
                    ssh ken3@192.168.2.70 "
                        docker pull ${params.DOCKERHUB_USER}/${params.PRODUCTSERVICE}:${params.IMAGE_TAG} &&
                        docker rm -f ${params.CONTAINER_PRODUCTSERVICE} || true &&
                        docker run -d -p 5003:5003 --name ${params.CONTAINER_PRODUCTSERVICE} ${params.DOCKERHUB_USER}/${params.PRODUCTSERVICE}:${params.IMAGE_TAG}
                    "
                """
            }
        }
      }
    }
}