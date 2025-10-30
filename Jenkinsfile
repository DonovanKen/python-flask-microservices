@Library('testsharedlibry') _


pipeline {
    agent any

    parameters {
       string(name: 'FRONTEND', defaultValue: 'frontend', description: 'this is my docker image name for frontend')
       string(name: 'ORDERSERVICE', defaultValue: 'orderservice', description: 'this is my docker image for order service')
       string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'this is my docker tag name')
       string(name: 'CONTAINER_FRONTEND', defaultValue: 'frontend_container', description: 'this is my docker container name for frontend')
       string(name: 'CONTAINER_ORDERSERVICE', defaultValue: 'orderservice_container', description: 'this is my docker container name for orderservice')
       string(name: 'GITHUB_USER', defaultValue: 'DonovanKen', description: 'user')
       string(name: 'DOCKERHUB_USER', defaultValue: 'mrkendono', description: 'gocker hub')


    }
    stages {
      stage('Build Image') {
        steps {
          script {
           dockerBuild("$FRONTEND", "$IMAGE_TAG")
           dockerBuild("$ORDERSERVICE", "$IMAGE_TAG")
          }
        }
      }

      stage('Run Container Test') {
        steps {
            script {
                testimage("$FRONTEND", "$IMAGE_TAG", "$CONTAINER_FRONTEND")
                testimage("$ORDERSERVICE", "$IMAGE_TAG", "$CONTAINER_ORDERSERVICE")
            }
        }
      }

      stage('Deploy') {
        environment {
            DOCKERHUB_USER = "mrkendono"
            FRONTEND          = "frontend"
            ORDERSERVICE      = "orderservice"
            IMAGE_TAG         = "latest"
            CONTAINER_FRONTEND    = "frontend_container"
            CONTAINER_ORDERSERVICE    = "frontend_container"
        }
        steps {
            //sshagent(credentials: ['ssh-cred']) {
              //  sh '''
                //    command1="docker pull $DOCKERHUB_USER/$FRONTEND:$IMAGE_TAG"
                //    command2="docker rm -f $CONTAINER_NAME"
                  //  command3="docker run -d -p 5000:5000 --name $CONTAINER_NAME $DOCKERHUB_USER/$FRONTEND:$IMAGE_TAG"
                  //  [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                  //  ssh-keyscan -t rsa,dsa 192.168.2.69 >> ~/.ssh/known_hosts
                  //  echo "testing..."
                   // ssh ub2@192.168.2.69 \
                  //      -o SendEnv=DOCKERHUB_USER \
                   //     -o SendEnv=FRONTEND \
                   //     -o SendEnv=IMAGE_TAG \
                     ////   -o SendEnv=CONTAINER_NAME \
                  //      -C "$command1 && $command2 && $command3"
                ////'''
               //}

               sshagent(credentials: ['ssh-cred']) {
                sh '''
                    command1="docker pull $DOCKERHUB_USER/$FRONTEND:$IMAGE_TAG"
                    command2="docker rm -f $CONTAINER_FRONTEND"
                    command3="docker run -d -p 5000:5000 --name $CONTAINER_FRONTEND $DOCKERHUB_USER/$FRONTEND:$IMAGE_TAG"
                    [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    ssh-keyscan -t rsa,dsa 192.168.2.70 >> ~/.ssh/known_hosts

                    ssh ken3@192.168.2.70 \
                        -o SendEnv=DOCKERHUB_USER \
                        -o SendEnv=FRONTEND \
                        -o SendEnv=IMAGE_TAG \
                        -o SendEnv=CONTAINER_FRONTEND \
                        -C "$command1 && $command2 && $command3"
                '''


                sh '''
                    command01="docker pull $DOCKERHUB_USER/$ORDERSERVICE:$IMAGE_TAG"
                    command02="docker rm -f $CONTAINER_ORDERSERVICE"
                    command03="docker run -d -p 5000:5000 --name $CONTAINER_ORDERSERVICE $DOCKERHUB_USER/$ORDERSERVICE:$IMAGE_TAG"
                    [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    ssh-keyscan -t rsa,dsa 192.168.2.70 >> ~/.ssh/known_hosts

                    ssh ken3@192.168.2.70 \
                        -o SendEnv=DOCKERHUB_USER \
                        -o SendEnv=ORDERSERVICE \
                        -o SendEnv=IMAGE_TAG \
                        -o SendEnv=CONTAINER_ORDERSERVICE \
                        -C "$command01 && $command02 && $command03"
                '''
               }
           }

      
               
        }
      }
}