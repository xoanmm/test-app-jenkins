pipeline {
  agent any
  stages {
    stage('Test') {
      steps {
        container('tools') {
          dir('src') {
            sh 'pip install --upgrade pip'
            sh 'pip3 install -r requirements.txt'
            sh 'python -m pytest --cov'
            sh 'cp coverage.xml ..'
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
