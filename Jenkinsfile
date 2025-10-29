pipeline {
    agent any
    environment {
        IMAGE_NAME = 'devsecops-lab-web'
    }

    stages {
        stage('Checkout') {
            steps {
                // More robust Git checkout
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker compose build'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Run tests inside the Docker container
                    sh 'docker compose run web pytest -v'
                }
            }
        }

        stage('Static Analysis (Bandit)') {
            steps {
                script {
                    // Run Bandit inside Docker; don't fail pipeline on warnings
                    sh 'docker compose run web bandit app.py || true'
                }
            }
        }

        stage('Dependency Scan (Safety)') {
            steps {
                script {
                    // Run Safety inside Docker; don't fail pipeline on warnings
                    sh 'docker compose run web safety check -r requirements.txt || true'
                }
            }
        }

        stage('Image Scan (Trivy)') {
            steps {
                script {
                    // Scan Docker image with Trivy; don't fail pipeline on warnings
                    sh "trivy image ${IMAGE_NAME} || true"
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh 'docker compose up -d'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished — cleaning up containers.'
            sh 'docker compose down || true'
        }
    }
}
