pipeline {
    agent any

    environment {
        DOCKER_BUILDKIT = 1 // Enable Docker BuildKit for faster builds
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/charlino/example-voting-app.git'
            }
        }

        stage('Build Services') {
            steps {
                sh 'docker-compose build --parallel'
            }
        }

        stage('Start Services') {
            steps {
                sh 'docker-compose up -d --remove-orphans'
            }
        }

        stage('Verify Health Checks') {
            steps {
                script {
                    def maxRetries = 5
                    def retryCount = 0
                    def healthy = false

                    while (retryCount < maxRetries) {
                        def redisId = sh(script: "docker-compose ps -q redis", returnStdout: true).trim()
                        def dbId = sh(script: "docker-compose ps -q db", returnStdout: true).trim()

                        if (!redisId || !dbId) {
                            echo "Error: Redis or DB container is not running!"
                            break
                        }

                        def redisHealth = sh(script: "docker inspect --format='{{.State.Health.Status}}' ${redisId}", returnStdout: true).trim()
                        def dbHealth = sh(script: "docker inspect --format='{{.State.Health.Status}}' ${dbId}", returnStdout: true).trim()

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
            }
        }
    }

    post {
        always {
            echo "Cleaning up Docker resources..."
            sh 'docker-compose down --volumes'
        }
        failure {
            echo "Pipeline failed! Check logs for errors."
        }
    }
}
