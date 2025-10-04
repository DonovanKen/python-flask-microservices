@Library('testsharedlibry') _


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
           dockerBuild("$IMAGE_NAME", "$IMAGE_TAG")
          }
        }
      }

      stage('Run Container Test') {
        steps {
            script {
                testimage("$IMAGE_NAME", "$IMAGE_TAG", "$CONTAINER_NAME")
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
                    echo "testing..."
                    ssh ub2@192.168.2.69 \
                        -o SendEnv=DOCKERHUB_USER \
                        -o SendEnv=IMAGE_NAME \
                        -o SendEnv=IMAGE_TAG \
                        -o SendEnv=CONTAINER_NAME \
                        -C "$command1 && $command2 && $command3"
                '''
               }

               sshagent(credentials: ['ssh-cred']) {
                sh '''
                    command1="docker pull $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                    command2="docker rm -f $CONTAINER_NAME"
                    command3="docker run -d -p 5000:5000 --name $CONTAINER_NAME $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                    [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    ssh-keyscan -t rsa,dsa 192.168.2.70 >> ~/.ssh/known_hosts

                    ssh ken3@192.168.2.70 \
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