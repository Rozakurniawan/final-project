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
        git branch: 'main', url: 'https://github.com/Rozakurniawan/final-project.git'
        Checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
      }
    }

    stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'your_dockerhub_id', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUsername')]) {
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'your_dockerhub_id') {
                            dockerImage.push("${env.BUILD_NUMBER}")
                            dockerImage.push('latest')
                        }
                    }
                }
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

