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
          - name: docker
            image: docker:19.03.1
            readinessProbe:
              exec:
                command: [sh, -c, "ls -S /var/run/docker.sock"]
            command:
            - sleep
            args:
            - 99d
            volumeMounts:
            - name: docker-socket
              mountPath: /var/run
          - name: docker-daemon
            image: docker:19.03.1-dind
            securityContext:
              privileged: true
            volumeMounts:
            - name: docker-socket
              mountPath: /var/run
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
        container('docker') {
          sh 'docker version && DOCKER_BUILDKIT=1 docker build --progress plain -t xoanmallon/test-app-jenkins:develop .'
        }
      }
    }
  }
}