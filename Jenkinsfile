pipeline {
    agent any

    environment {
        IMAGE_NAME = 'dedeepya200/week12'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

         stage('Run Selenium Tests with pytest') {
            steps {
                echo "Running Selenium Tests using pytest"

                // Install Python dependencies
                sh 'pip3 install -r requirements.txt'

                // Start Flask application in background
                sh 'python app.py &'

                // Wait for Flask application to start
                sh 'ping 127.0.0.1 -n 5 > nul'

                // Run Selenium tests
                sh 'python3 -m pytest -v'
            }
        }


        stage('Build Docker Image') {
            steps {
                echo "Building Flask Docker Image: ${IMAGE_NAME}:${IMAGE_TAG}"

                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Login') {
            steps {
                echo "Logging in to Docker Hub"

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

        stage('Push Docker Image') {
            steps {
                echo "Pushing ${IMAGE_NAME}:${IMAGE_TAG}"

                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Check Kubernetes Connection') {
            steps {
                echo "Checking Kubernetes Connection"

                sh 'kubectl config current-context'
                sh 'kubectl get nodes'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying Flask Application"

                sh 'kubectl apply -f service.yaml'

                sh """
                    kubectl set image deployment/flaskapp \
                    flaskapp=${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Checking Kubernetes Resources"

                sh 'kubectl rollout status deployment/flaskapp'
                sh 'kubectl get deployments'
                sh 'kubectl get pods'
                sh 'kubectl get services'
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }

        failure {
            echo "Pipeline failed. Please check the logs."
        }
    }
}

