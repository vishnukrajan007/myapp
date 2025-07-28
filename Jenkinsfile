pipeline {
    agent any

    environment {
        IMAGE_NAME = 'vishnukrajan007/myapp:latest'
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

        stage('Deploy') {
            steps {
                sh '''
                    docker pull $IMAGE_NAME
                    docker stop myapp || true
                    docker rm myapp || true
                    docker run -d --name myapp -p 80:80 $IMAGE_NAME
                '''
            }
        }
    }
}
