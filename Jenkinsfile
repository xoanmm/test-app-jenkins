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
            image: docker:23.0.4-dind
            tty: true
            securityContext:
              allowPrivilegeEscalation: false
              runAsUser: 0
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