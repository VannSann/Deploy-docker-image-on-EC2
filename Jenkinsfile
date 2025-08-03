pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'VannSann/my_java-application'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        REMOTE_HOST = 'ec2-user@44.202.86.49'
    }

    stages {
        stage('Clone Code') {
            steps {
                git url: 'https://github.com/VannSann/Deploy-docker-image-on-EC2.git', branch: 'main'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$IMAGE_TAG .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no $REMOTE_HOST '
                      docker pull $DOCKER_IMAGE:$IMAGE_TAG &&
                      docker stop flaskapp || true &&
                      docker rm flaskapp || true &&
                      docker run -d --name flaskapp -p 80:5000 $DOCKER_IMAGE:$IMAGE_TAG
                    '
                '''
            }
            }
    }
}
