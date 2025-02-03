pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/charlino/example-voting-app.git'
            }
        }

        stage('Build Services') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Start Services') {
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Verify Health Checks') {
            steps {
                script {
                    def maxRetries = 5
                    def retryCount = 0
                    def healthy = false
                    
                    while (retryCount < maxRetries) {
                        def redisHealth = sh(script: "docker inspect --format='{{.State.Health.Status}}' $(docker-compose ps -q redis)", returnStdout: true).trim()
                        def dbHealth = sh(script: "docker inspect --format='{{.State.Health.Status}}' $(docker-compose ps -q db)", returnStdout: true).trim()
                        
                        if (redisHealth == "healthy" && dbHealth == "healthy") {
                            echo "All services are healthy!"
                            healthy = true
                            break
                        }

                        echo "Waiting for services to be healthy... Attempt ${retryCount + 1}/${maxRetries}"
                        sleep 10
                        retryCount++
                    }

                    if (!healthy) {
                        error "Services did not become healthy in time!"
                    }
                }