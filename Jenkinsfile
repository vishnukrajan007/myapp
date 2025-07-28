pipeline {
  agent any

  environment {
    IMAGE_NAME = 'vishnukrajan007/frontend'
    IMAGE_TAG = "${env.GIT_COMMIT}"
    FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    LATEST_IMAGE = "${IMAGE_NAME}:latest"
    AWS_REGION = 'ap-south-1'
    EKS_CLUSTER_NAME = 'vkr-cluster'
  }

  stages {
    stage('Clone') {
      steps {
        git branch: 'main',
            url: 'https://github.com/vishnukrajan007/myapp.git',
            credentialsId: 'github-token'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh """
          docker build --pull --no-cache -t ${FULL_IMAGE} -t ${LATEST_IMAGE} -f dockerfiles/Dockerfile.frontend .
        """
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                          usernameVariable: 'DB_USER',
                                          passwordVariable: 'DB_PASS')]) {
          sh """
            echo "$DB_PASS" | docker login -u "$DB_USER" --password-stdin
            docker push ${FULL_IMAGE}
            docker push ${LATEST_IMAGE}
          """
        }
      }
    }

    stage('Deploy to EKS') {
      steps {
        withCredentials([
          string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
          string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
        ]) {
          sh """
            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
            aws configure set default.region $AWS_REGION

            aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION

            kubectl set image deployment/frontend-deployment frontend=${FULL_IMAGE} --record
            kubectl rollout status deployment/frontend-deployment
          """
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}


