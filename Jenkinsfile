pipeline {
  agent any
  stages {
    stage('Test') {
      steps {
        container('tools') {
          dir('src') {
            sh 'pip3 install --upgrade pip3'
            sh 'pip3 install -r requirements.txt'
            sh 'pytest --cov'
          }
        }
      }
    }
    // stage('Build') {
    //   steps {
    //     container('docker') {
    //       sh 'docker version && DOCKER_BUILDKIT=1 docker build --progress plain -t xoanmallon/test-app-jenkins:develop .'
    //     }
    //   }
    // }
  }
}
