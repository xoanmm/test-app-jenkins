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
          - name: docker-client
            image: docker:19.03.1
            command: ['sleep', '99d']
            env:
              - name: DOCKER_HOST
                value: tcp://localhost:2375
          - name: docker-daemon
            image: docker:19.03.1-dind
            env:
              - name: DOCKER_TLS_CERTDIR
                value: ""
            securityContext:
              privileged: true
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
        container('docker-client') {
          sh 'docker build -t xoanmallon/test-app-jenkins:develop .'
        }
      }
    }
  }
}