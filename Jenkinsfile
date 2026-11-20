pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE = 'sarradraouil/monapp'
    TAG   = "build-${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Docker Build') {
      steps {
        bat 'docker version'
        bat "docker build -t %IMAGE%:%TAG% ."
      }
    }

    stage('Smoke Test') {
      steps {
        bat """
          docker rm -f monapp_test 2>nul || ver > nul
          docker run -d --name monapp_test -p 8090:80 %IMAGE%:%TAG%
          ping -n 3 127.0.0.1 > nul
          curl -I http://localhost:8090 | find "200 OK"
          docker rm -f monapp_test
        """
      }
    }

    stage('Push (Docker Hub)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                          usernameVariable: 'USER',
                                          passwordVariable: 'PASS')]) {
          bat """
            docker login -u %USER% -p %PASS%
            docker tag %IMAGE%:%TAG% %IMAGE%:latest
            docker push %IMAGE%:%TAG%
            docker push %IMAGE%:latest
          """
        }
      }
    }
  }

  post {
    success { echo 'Build+Test+Push OK' }
    failure { echo 'Build/Tests/Push KO' }
  }
}
