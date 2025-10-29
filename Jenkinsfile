pipeline {
    agent any
    environment {
        IMAGE_NAME = 'devsecops-lab-web'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out repository..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh 'sudo docker compose build'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo "Running tests inside Docker container..."
                    sh 'sudo docker compose run web pytest -v'
                }
            }
        }

        stage('Static Analysis (Bandit)') {
            steps {
                script {
                    echo "Running Bandit security scan..."
                    def banditExit = sh(script: 'sudo docker compose run web bandit -r app.py -f html -o bandit-report.html', returnStatus: true)

                    // Archive the Bandit report
                    archiveArtifacts artifacts: 'bandit-report.html', fingerprint: true

                    if (banditExit != 0) {
                        echo "Bandit found issues! Check bandit-report.html."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Dependency Scan (Safety)') {
            steps {
                script {
                    echo "Running Safety dependency scan..."
                    def safetyExit = sh(script: 'sudo docker compose run web safety check -r requirements.txt || true', returnStatus: true)
                    
                    if (safetyExit != 0) {
                        echo "Safety found vulnerabilities."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Image Scan (Trivy)') {
            steps {
                script {
                    echo "Scanning Docker image with Trivy..."
                    def trivyExit = sh(script: "sudo trivy image ${IMAGE_NAME} || true", returnStatus: true)

                    if (trivyExit != 0) {
                        echo "Trivy found vulnerabilities."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    echo "Deploying application..."
                    sh 'sudo docker compose up -d'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished — cleaning up Docker containers...'
            sh 'sudo docker compose down || true'
            cleanWs()
        }
    }
}
