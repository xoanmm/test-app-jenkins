pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: python
            image: python:3.8
            tty: true
          - name: dind
            image: 23.0.4-git
            command: [""]
            tty: true
        '''
    }
  }
  stages {
    stage('Test') {
      steps {
        container('python') {
          dir('src') {
            sh 'pip3 install -r requirements.txt'
            sh 'pytest --cov'
          }
        }
      }
    }
    stage('Build') {
      steps {
        container('dind') {
          sh 'docker build -t xoanmallon/test-app-jenkins:develop .'
        }
      }
    }
  }
}