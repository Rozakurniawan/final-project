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
          def dockerhubCreds = 'dckr_pat_HEYpGtP2JyG7im-QPA3ibGkRdHQ'

          withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
            sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
          }
        }
      }
    }

    stage('List pods') {
    withKubeConfig([credentialsId: 'kubernetes-config']) {
        sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
        sh 'chmod u+x ./kubectl'  
        sh './kubectl get pods'
    }
  }
    
    stage('Deploy to Dev') {
      steps {
        sh 'kubectl apply -f namespace.yaml'
        sh 'kubectl apply -f dev-configmap.yaml'
        sh 'kubectl apply -f dev-secret.yaml'
        sh 'helm upgrade --install flask-dev ./helm-chart --namespace dev'
      }
    }
  }
}
