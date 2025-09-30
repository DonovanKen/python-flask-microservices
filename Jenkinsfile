pipeline {
    agent non

parameters {
    string(name: 'IMAGE_NAME', defaultValue: 'frontend', description: 'this is my docker image name')
    string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'this is my docker tag name')
    string(name: 'CONTAINER_NAME', defaultValue: 'frontend_container', description: 'this is my docker container name')
    string(name: 'GITHUB_USER', defaultValue: 'DonovanKen', description: '')
    string(name: 'DOCKER_HUB_USER', defaultValue: 'mrkendono', description: '')


}

stage {
    stage('Build Image')
      steps {
        script {
            sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
        }
      }
}
