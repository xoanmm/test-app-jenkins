stage('test') {
     agent {
          docker {
               image 'python:3.8'
          }
     }
     steps {
          sh 'pip install -r requirements.txt'
          sh 'pytest'
     }
}