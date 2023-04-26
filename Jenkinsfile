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
            sh 'pytest --cov --junitxml=reports/output.xml'
            sh 'ls -lh'
          }
        }
      }
      post {
        always {
            // Archive unit tests for the future
            always{
                step([$class: 'CoberturaPublisher',
                                autoUpdateHealth: false,
                                autoUpdateStability: false,
                                coberturaReportFile: 'src/reports/output.xml',
                                failNoReports: false,
                                failUnhealthy: false,
                                failUnstable: false,
                                maxNumberOfBuilds: 10,
                                onlyStable: false,
                                sourceEncoding: 'ASCII',
                                zoomCoverageChart: false])
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