pipeline {
    agent any

    parameters {
       string(name: 'IMAGE_NAME', defaultValue: 'frontend', description: 'this is my docker image name')
       string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'this is my docker tag name')
       string(name: 'CONTAINER_NAME', defaultValue: 'frontend_container', description: 'this is my docker container name')
       string(name: 'GITHUB_USER', defaultValue: 'DonovanKen', description: 'user')
       string(name: 'DOCKERHUB_USER', defaultValue: 'mrkendono', description: 'gocker hub')


    }
    stages {
      stage('Build Image') {
        steps {
          script {
            sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} frontend/'
          }
        }
      }

      stage('Run Container Test') {
        steps {
            script {
                sh '''
                   echo "delete old container and create new..."
                   docker rm -f ${CONTAINER_NAME}
                   docker run -d -p 5000:5000 --name ${CONTAINER_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
                   docker ps
                   sleep 5

                   echo "testing container..."
                   curl -I http://$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${CONTAINER_NAME}):5000

                   echo "delete tested container..."
                   docker rm -f ${CONTAINER_NAME}
                '''
            }
        }
      }

      stage('Deploy') {
        environment {
            DOCKERHUB_USER = "mrkendono"
            IMAGE_NAME      = "frontend"
            IMAGE_TAG       = "latest"
            CONTAINER_NAME  = "frontend_container"
        }
        steps {
            sshagent(credentials: ['ssh-cred']) {
                sh '''
                    command1="docker pull $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                    command2="docker rm -f $CONTAINER_NAME"
                    command3="docker run -d -p 5000:5000 --name $CONTAINER_NAME $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                    [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    ssh-keyscan -t rsa,dsa 192.168.2.69 >> ~/.ssh/known_hosts

                    ssh ub2@192.168.2.69 \
                        -o SendEnv=DOCKERHUB_USER \
                        -o SendEnv=IMAGE_NAME \
                        -o SendEnv=IMAGE_TAG \
                        -o SendEnv=CONTAINER_NAME \
                        -C "$command1 && $command2 && $command3"
                '''
               }
           }

      
               
        }
      }
}