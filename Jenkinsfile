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
          - name: helm
            image: alpine/helm:3.8.2
            command:
            - sleep
            args:
            - 99d
          serviceAccountName: jenkins-ci
          volumes:
          - name: dockersock
            hostPath:
              path: /var/run/docker.sock
        '''
    }
  }
  stages {
  //   stage('Test') {
  //     steps {
  //       container('python') {
  //         dir('src') {
  //           sh 'pip3 install -r requirements.txt'
  //           sh 'pytest --cov --cov-report xml'
  //           sh 'cp coverage.xml ..'
  //           sh 'ls -lh'
  //         }
  //       }
  //     }
  //     post {
  //       always {
  //           // Archive unit tests for the future
  //           step([$class: 'CoberturaPublisher',
  //                                 autoUpdateHealth: false,
  //                                 autoUpdateStability: false,
  //                                 coberturaReportFile: 'src/coverage.xml',
  //                                 failNoReports: false,
  //                                 failUnhealthy: false,
  //                                 failUnstable: false,
  //                                 maxNumberOfBuilds: 10,
  //                                 onlyStable: false,
  //                                 sourceEncoding: 'ASCII',
  //                                 zoomCoverageChart: false])
  //       }
  //     }
  //   }
  //   stage('Build') {
  //     steps {
  //       container('docker') {
  //         sh 'docker version && DOCKER_BUILDKIT=1 docker build --progress plain -t xoanmallon/test-app-jenkins:develop .'
  //       }
  //     }
  //   }
    stage('Deploy') {
      steps {
        container('helm') {
          sh 'helm repo add bitnami https://charts.bitnami.com/bitnami'
          sh 'helm repo update'
          sh 'helm dep up helm'
          sh 'helm -n fast-api upgrade -i my-app helm --create-namespace --wait'
        }
      }
    }
  }
}