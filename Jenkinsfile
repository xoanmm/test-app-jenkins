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
            env:
            - name: REGISTRY_TOKEN
              valueFrom:
                secretKeyRef:
                  name: reg-creds
                  key: REGISTRY_TOKEN
            - name: REGISTRY_USERNAME
              valueFrom:
                secretKeyRef:
                  name: reg-creds
                  key: REGISTRY_USERNAME
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
  environment {
    GITHUB_TOKEN  = credentials('github_app_token')
    DOCKER_IMAGE_REPOSITORY = "xoanmallon/test-app-jenkins"
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
  stage('Release') {
      when {
        branch 'test'
      }
      steps {
        nodejs(nodeJSInstallationName: 'node-20.1.0') {
          sh 'npm config ls'
          sh 'npm install'
          sh 'npx semantic-release'
          script {
            version = readFile('semantic_release_version.txt').trim()
          }
          script {
            if (version != "") {
              sh "echo 'Version generated using semantic-release is ${version}!'"
            } else {
              echo 'No version generated using semantic-release'
            }
          }
        }
      }
    }
    stage('Docker build & push') {
      when {
        branch 'test'
        expression { version != '' }
      }
      steps {
        container('docker') {
          sh 'echo $REGISTRY_TOKEN | docker login -u $REGISTRY_USERNAME --password-stdin'
          sh "docker version && DOCKER_BUILDKIT=1 docker build --progress plain -t ${env.DOCKER_IMAGE_REPOSITORY}:${version} ."
          sh "docker push ${env.DOCKER_IMAGE_REPOSITORY}:${version}"
        }
      }
    }
    stage('Deploy') {
      when {
        branch 'test'
        expression { version != '' }
      }
      steps {
        container('helm') {
          echo "Version to deploy is ${version}"
          sh 'helm repo add bitnami https://charts.bitnami.com/bitnami'
          sh 'helm repo update'
          sh 'helm dep up helm'
          sh 'helm plugin install https://github.com/databus23/helm-diff --version 3.7.0'
          sh "helm diff upgrade -n fast-api my-app helm --set image.repository=${env.DOCKER_IMAGE_REPOSITORY} --set image.tag=${version} --allow-unreleased"
          script {
            input message: "Apply the helm changes?"
          }
          sh 'helm -n fast-api upgrade -i my-app helm --create-namespace --wait'
        }
      }
    }
  }
}