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
              privileged: true
            env:
            - name: DOCKER_TLS_CERTDIR
              value: ""
            - name: DOCKER_HOST
              value: tcp://localhost:2375
            volumeMounts:
            - name: cache
              mountPath: /var/lib/docker
        volumes:
          - name: cache
            hostPath:
              path: /tmp
              type: Directory
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