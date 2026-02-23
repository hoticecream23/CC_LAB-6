pipeline {
    agent any

    parameters {
        string(name: 'BACKEND_COUNT', defaultValue: '1', description: 'Number of backend containers (1 or 2)')
    }

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend image..."
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Deploying backend containers..."
                
                docker network create app-network || true
                docker rm -f backend1 backend2 || true

                if [ "$BACKEND_COUNT" = "1" ]; then
                    echo "Running 1 backend container"
                    docker run -d --name backend1 --network app-network backend-app
                else
                    echo "Running 2 backend containers"
                    docker run -d --name backend1 --network app-network backend-app
                    docker run -d --name backend2 --network app-network backend-app
                fi
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Starting NGINX..."

                docker rm -f nginx-lb || true

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                docker cp $WORKSPACE/nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }

        stage('Verification') {
            steps {
                sh '''
                echo "Running containers:"
                docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline SUCCESS: Backend and NGINX are running.'
        }
        failure {
            echo 'Pipeline FAILED: Check console output.'
        }
    }
}