pipeline {
    agent any
    environment {
        IMAGE_NAME = 'python-devsecops-jenkins_app'
    }
    stages {
        stage('Checkout') {
            steps {
                // Pull the code from GitHub
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    // Create virtual environment
                    sh 'python3 -m venv venv'
                    // Upgrade pip
                    sh './venv/bin/pip install --upgrade pip'
                    // Install Python dependencies + dev/security tools
                    sh './venv/bin/pip install -r requirements.txt bandit safety'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Run the tests with pytest
                    sh './venv/bin/pytest -v'
                }
            }
        }

        stage('Static Code Analysis (Bandit)') {
            steps {
                script {
                    // Run Bandit for static code analysis
                    sh './venv/bin/bandit -r .'
                }
            }
        }

        stage('Check Dependency Vulnerabilities (Safety)') {
            steps {
                script {
                    // Run Safety to check Python dependencies
                    sh './venv/bin/safety check'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image once
                    sh 'docker-compose build'
                }
            }
        }

        stage('Container Vulnerability Scan (Trivy)') {
            steps {
                script {
                    // Scan the Docker image with Trivy
                    sh "trivy image ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    // Deploy the application using Docker Compose
                    sh 'docker-compose up -d'
                }
            }
        }
    }
    post {
        always {
            // Clean workspace after build
            cleanWs()
        }
    }
}
