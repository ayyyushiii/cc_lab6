pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend image..."

                # Remove old image if exists
                docker rmi -f backend-app || true

                # Build backend docker image
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Deploying backend containers..."

                # Remove old containers
                docker rm -f backend1 backend2 || true

                # Run backend containers
                docker run -d --name backend1 backend-app
                docker run -d --name backend2 backend-app

                # Wait for containers to register in Docker DNS
                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Starting NGINX load balancer..."

                # Remove old nginx container
                docker rm -f nginx-lb || true

                # Run nginx container
                docker run -d --name nginx-lb -p 80:80 nginx

                # Wait for nginx startup
                sleep 3

                # Copy nginx configuration
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Small delay before reload
                sleep 2

                # Reload nginx
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check console logs."
        }
    }
}