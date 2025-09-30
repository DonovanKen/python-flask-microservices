pipeline {
    agent any

    parameters {
       string(name: 'IMAGE_NAME', defaultValue: 'frontend', description: 'this is my docker image name')
       string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'this is my docker tag name')
       string(name: 'CONTAINER_NAME', defaultValue: 'frontend_container', description: 'this is my docker container name')
       string(name: 'GITHUB_USER', defaultValue: 'DonovanKen', description: 'user')
       string(name: 'DOCKER_HUB_USER', defaultValue: 'mrkendono', description: 'gocker hub')


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
      stage('push Image') {
        steps {
            script {
                withCredentials([usernamePassword(
                credentialsId: 'github_credentials', 
                passwordVariable: 'DOCKERHUB_PASSWORD', 
                usernameVariable: 'DOCKERHUB_USER')]) {
    // some block
                sh '''
                   docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_PASSWORD}/${IMAGE_NAME}:${IMAGE_TAG}
                   docker login -u DOCKERHUB_USER -p DOCKERHUB_PASSWORD
                   docker push ${DOCKERHUB_USER}/{IMAGE_NAME}:${IMAGE_TAG}

                '''
                }


            }
        }
      }

        
    }

}