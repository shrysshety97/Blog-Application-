pipeline {

    agent any

    environment {
        DOCKER_IMAGE_BACKEND = "shreyasdocs/blog-backend:latest"
        DOCKER_IMAGE_FRONTEND = "shreyasdocs/blog-frontend:latest"
        EC2_HOST = "13.203.199.8"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                dir('BlogApplication') {
                    sh './mvnw clean package -Dmaven.test.skip=true'
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE_BACKEND ./BlogApplication'
                sh 'docker build -t $DOCKER_IMAGE_FRONTEND ./app'
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'docker push $DOCKER_IMAGE_BACKEND'
                sh 'docker push $DOCKER_IMAGE_FRONTEND'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$EC2_HOST << 'EOF'

                        docker pull shreyasdocs/blog-backend:latest
                        docker pull shreyasdocs/blog-frontend:latest

                        docker rm -f blog-backend || true
                        docker rm -f blog-frontend || true

                        docker run -d \
                          --name blog-backend \
                          --network blog-network \
                          -p 8085:8085 \
                          --env-file ~/blog.env \
                          shreyasdocs/blog-backend:latest

                        docker run -d \
                          --name blog-frontend \
                          --network blog-network \
                          -p 80:80 \
                          shreyasdocs/blog-frontend:latest

                        EOF
                    '''
                }
            }
        }
    }
}