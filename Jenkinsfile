pipeline {
  agent any

  environment {
    IMAGE_NAME = "texsa/flask-app"
    IMAGE_TAG = "latest"
    NAMESPACE = "dev"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
            sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
          }
        }
      }
    }
    
    stage('Deploy to Dev') {
      steps {
        withKubeConfig{
        
        sh '/usr/local/bin/kubectl apply -f namespace.yaml'
        sh '/usr/local/bin/kubectl apply -f dev-configmap.yaml'
        sh '/usr/local/bin/kubectl apply -f dev-secret.yaml'
        sh 'helm upgrade --install flask-dev ./helm-chart --namespace dev'
        }
      }
    }
  }
}
