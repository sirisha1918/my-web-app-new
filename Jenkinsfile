pipeline {
    agent any
 
    environment {
        DOCKER_IMAGE = "advanced-web-app:1.0"
        DOCKER_HUB_USER = 'sudheer99123' // Replace with your Docker Hub username
        DOCKER_HUB_CREDENTIALS = 'Sudheer@docker' // Jenkins credentials ID
    }
 
    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from SCM...'
                checkout scm
            }
        }
 
        stage('Build Application') {
            steps {
                echo 'Building the application with Maven...'
                sh 'mvn clean package'
            }
        }
 
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }
 
        stage('Test Docker Image') {
            steps {
                echo 'Testing the Docker container...'
                sh "docker run -d -p 8080:8080 --name test-container ${DOCKER_IMAGE}"
                // Optionally add HTTP-based testing here
                sh "docker stop test-container && docker rm test-container"
            }
        }
 
        stage('Push Docker Image to Registry') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker tag ${DOCKER_IMAGE} ${DOCKER_HUB_USER}/${DOCKER_IMAGE}"
                    sh "docker push ${DOCKER_HUB_USER}/${DOCKER_IMAGE}"
                }
            }
        }
 
        stage('Deploy Application') {
            steps {
                echo 'Deploying application using Docker...'
                sh "docker run -d -p 8080:8080 ${DOCKER_HUB_USER}/${DOCKER_IMAGE}"
            }
        }
    }
 
    post {
        always {
            echo 'Cleaning up Docker containers and images...'
            sh "docker rm -f $(docker ps -aq) || true"
            sh "docker rmi $(docker images -q) || true"
        }
    }
}