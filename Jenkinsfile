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
          - name: kaniko
            image: gcr.io/kaniko-project/executor:v1.9.0-debug
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
        container('kaniko') {
          sh '/kaniko/executor --context . --dockerfile "Dockerfile"'
        }
      }
    }
  }
}