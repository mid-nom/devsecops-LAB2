pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Setting up Python virtual environment and installing dependencies...'
                script {
                    sh 'python3 -m venv venv'
                    sh './venv/bin/pip install -r requirements.txt'
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests inside Docker...'
                // Using docker-compose instead of docker compose
                sh 'docker-compose run --rm web pytest -v'
            }
        }

        stage('Static Analysis (Bandit)') {
            steps {
                echo 'Running Bandit static analysis...'
                sh 'docker-compose run --rm web bandit app.py || true'
            }
        }

        stage('Dependency Scan (Safety)') {
            steps {
                echo 'Scanning dependencies with Safety...'
                sh 'docker-compose run --rm web safety scan -r requirements.txt || true'
            }
        }

        stage('Image Scan (Trivy)') {
            steps {
                echo 'Scanning Docker image with Trivy...'
                sh 'trivy image devsecops-lab-web || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker-compose build'
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying application...'
                sh 'docker-compose up -d || true'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished — cleaning up containers.'
            sh 'docker-compose down || true'
        }
    }
}
