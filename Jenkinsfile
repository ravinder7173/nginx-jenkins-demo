pipeline {
  agent any
  environment {
    IMAGE = "nginx-jenkins:${BUILD_NUMBER}"
    CONTAINER_NAME = "nginx-jenkins"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t $IMAGE .'
      }
    }

    stage('Stop & Remove old container (if any)') {
      steps {
        sh '''
          if docker ps -q -f name=$CONTAINER_NAME >/dev/null 2>&1; then
            docker stop $CONTAINER_NAME || true
          fi
          if docker ps -a -q -f name=$CONTAINER_NAME >/dev/null 2>&1; then
            docker rm $CONTAINER_NAME || true
          fi
        '''
      }
    }

    stage('Run container') {
      steps {
        sh 'docker run -d --name $CONTAINER_NAME -p 80:80 $IMAGE'
      }
    }

    stage('Smoke test') {
      steps {
        sh 'sleep 3; curl -f http://localhost || true'
      }
    }
  }
  post {
    failure {
      sh 'echo "Build failed. Printing container status"; docker ps -a || true'
      sh 'docker logs $CONTAINER_NAME || true'
    }
  }
}

