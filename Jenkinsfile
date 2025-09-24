pipeline {
  agent any

  environment {
    IMAGE = "mitesh-patidar/clientapi:latest"
    NAMESPACE = "deltacapita"
  }

  stages {
    stage('Checkout') {
      steps { 
          checkout scm 
      }
    }

    stage('Test') {
            steps {
                script {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
     }

    stage('Build & Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDS', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            echo "$PASS" | docker login -u "$USER" --password-stdin
            docker build -t $IMAGE .
            docker push $IMAGE
          '''
        }
      }
    }

    stage('Deploy to K8s') {
      steps {
        withCredentials([file(credentialsId: 'KUBECONFIG', variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=$KUBECONFIG_FILE
            kubectl -n $NAMESPACE set image deployment/clientapi clientapi=$IMAGE --record || kubectl -n $NAMESPACE apply -f kubernetes_yamls/
          '''
        }
      }
    }
  }
}

