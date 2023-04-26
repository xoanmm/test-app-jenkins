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
            sh 'pytest --cov --cov-report xml'
            sh 'ls -lh'
            sh 'echo "GIT_URL: ${env.GIT_URL}"'
            script {
              currentBuild.result = 'SUCCESS'
            }
            step([$class: 'CompareCoverageAction', publishResultAs: 'statusCheck', scmVars: [GIT_URL: env.GIT_URL]])
          }
        }
      }
      post {
        always {
            // Archive unit tests for the future
            step([$class: 'CoberturaPublisher',
                                  autoUpdateHealth: false,
                                  autoUpdateStability: false,
                                  coberturaReportFile: 'src/coverage.xml',
                                  failNoReports: false,
                                  failUnhealthy: false,
                                  failUnstable: false,
                                  maxNumberOfBuilds: 10,
                                  onlyStable: false,
                                  sourceEncoding: 'ASCII',
                                  zoomCoverageChart: false])
            script {
              // if we are in a PR
              if (env.CHANGE_ID) {
                  publishCoverageGithub(filepath:'src/coverage.xml', coverageXmlType: 'cobertura', comparisonOption: [ value: 'optionFixedCoverage', fixedCoverage: '0.65' ], coverageRateType: 'Line')
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