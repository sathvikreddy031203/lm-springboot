pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = 'sathvikreddy031203/lm-springboot'   // <-- change
  }

  //triggers {
    // Webhook will trigger via GitHub; having pollSCM off is fine
 // }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build JAR') {
      steps {
        sh 'mvn -DskipTests clean package'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def tag = "${env.BUILD_NUMBER}"
          sh "docker build -t ${DOCKERHUB_REPO}:${tag} ."
          sh "docker tag ${DOCKERHUB_REPO}:${tag} ${DOCKERHUB_REPO}:latest"
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
          sh 'docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}'
          sh 'docker push ${DOCKERHUB_REPO}:latest'
        }
      }
    }

    stage('Deploy (run container on EC2)') {
      steps {
        script {
          // Stop & remove old container if exists
          sh 'docker rm -f springboot-demo || true'
          // Run new container mapping host 8081 -> container 8080
          sh 'docker run -d --name springboot-demo -p 8081:8080 ${DOCKERHUB_REPO}:latest'
        }
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
  }
}