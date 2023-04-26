def check_runs = new com.functions.buildGithubCheckScript()

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
            command:
            - sleep
            args:
            - 99d
            volumeMounts:
            - name: dockersock
              mountPath: /var/run/docker.sock
          volumes:
          - name: dockersock
            hostPath:
              path: /var/run/docker.sock
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
    stage("Unit Test") {
      steps {
        container('python') {
          dir('src') {
            script {
              sh 'pip3 install -r requirements.txt'
              try {
                def test = sh(script: "pytest --cov", returnStdout: true)
                check_runs.buildGithubCheck('success', "unit-test")
              } catch(Exception e) {
                check_runs.buildGithubCheck('failure', "unit-test")
                echo "Exception: ${e}"
              }
            }
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