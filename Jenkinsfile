pipeline {
    agent any

    environment {
        IMAGE_NAME = 'vishnukrajan007/myapp:latest'
        AWS_REGION = 'ap-south-1'
        EKS_CLUSTER_NAME = 'vkr-cluster'
    }

    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/vishnukrajan007/myapp.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME -f dockerfiles/Dockerfile.frontend .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_REGION

                        aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION

                        kubectl set image deployment/frontend-deployment myapp-container=$IMAGE_NAME --record
                        kubectl rollout status deployment/frontend-deployment
                    '''
                }
            }
        }
    }
}
