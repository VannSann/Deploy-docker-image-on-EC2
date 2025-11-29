pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'vannsann/my_java-application'
        SONARQUBE_ENV = credentials('sonarqube-token')
        IMAGE_TAG = ''
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/VannSann/Deploy-docker-image-on-EC2.git', branch: 'main', credentialsId: 'github-credentials'
                script {
                    env.GIT_COMMIT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    }
                    echo "Commit: ${env.GIT_COMMIT}"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                      sonar-scanner \
                      -Dsonar.projectKey=myapp \
                      -Dsonar.sources=. \
                      -Dsonar.java.binaries=target \
                      -Dsonar.login=$SONARQUBE_ENV
                    """
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Unit Tests') {
            steps {
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package'){
            steps{
                mvn clean package -DskipTests
            }
        }

        stage ('Archive Artifacts') {
            steps {
                script {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, excludes: '**/*.log'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                env.IMAGE_TAG = env.GIT_COMMIT
                sh 'docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh """
                 trivy image --format json --output trivy-image.json ${DOCKER_IMAGE}:${IMAGE_TAG}
                 trivy image --format table --output trivy-image.txt ${DOCKER_IMAGE}:${IMAGE_TAG}
                 trivy fs --format json --output trivy-fs.json .
                """
            }
        }

        stage('Generate Report') {
            steps {
                sh """
                echo "<html><body>" > report.html
                echo "<h2>Build #${BUILD_NUMBER} - Security & Quality Report</h2>" >> report.html
                echo "<h3>SonarQube Status: PASSED</h3>" >> report.html
                echo "<h3>Trivy Image Scan Summary</h3>" >> report.html
                trivy image --format table ${DOCKER_IMAGE} >> report.html
                echo "</body></html>" >> report.html
                """
            }
        }
    }
/*
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
                 ##ssh -o StrictHostKeyChecking=no $REMOTE_HOST '
                 docker pull $DOCKER_IMAGE:$IMAGE_TAG &&
                 docker stop springboot-app || true &&
                 docker rm springboot-app || true &&
                 docker run -d --name springboot-app -p 80:8080 $DOCKER_IMAGE:$IMAGE_TAG
                ##'
                '''
            }
        }

    }
    */
        post {
        always {
            cleanWs()
        }
    }
}
