pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Marypearl/example-voting-app.git'
            }
        }

        stage('Build Services') {
            steps {
                // Use the official Docker Compose image to invoke docker-compose commands
                sh 'docker run --rm -v $(pwd):/workspace -w /workspace docker/compose:1.29.2 -f docker-compose.yml build'
            }
        }

        stage('Start Services') {
            steps {
                sh 'docker run --rm -v $(pwd):/workspace -w /workspace docker/compose:1.29.2 -f docker-compose.yml up -d'
            }
        }

        stage('Verify Health Checks') {
            steps {
                script {
                    def maxRetries = 5
                    def retryCount = 0
                    def healthy = false
                    
                    while (retryCount < maxRetries) {
                        def redisHealth = sh(script: "docker inspect --format='{{.State.Health.Status}}' \$(docker-compose ps -q redis)", returnStdout: true).trim()
                        def dbHealth = sh(script: "docker inspect --format='{{.State.Health.Status}}' \$(docker-compose ps -q db)", returnStdout: true).trim()
                        
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

        stage('Run Seed Data (Optional)') {
            when {
                expression { params.RUN_SEED }
            }
            steps {
                sh 'docker run --rm -v $(pwd):/workspace -w /workspace docker/compose:1.29.2 -f docker-compose.yml --profile seed up -d'
            }
        }

        stage('Clean Up') {
            steps {
                sh 'docker run --rm -v $(pwd):/workspace -w /workspace docker/compose:1.29.2 -f docker-compose.yml down'
            }
        }
    }

    parameters {
        booleanParam(name: 'RUN_SEED', defaultValue: false, description: 'Run the seed data container')
    }
}