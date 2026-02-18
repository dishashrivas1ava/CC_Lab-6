pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create network if it doesn't exist
                docker network inspect app-network >/dev/null 2>&1 || \
                docker network create app-network

                # Remove old containers safely
                docker rm -f backend1 backend2 || true

                # Run backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Remove old nginx container
                docker rm -f nginx-lb || true

                # Run nginx and mount entire nginx folder
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  -v $(pwd)/nginx:/etc/nginx/conf.d \
                  nginx
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}

