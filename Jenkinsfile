pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/mid-nom/devsecops-LAB2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'docker compose run web pytest -v'
            }
        }

        stage('Static Analysis (Bandit)') {
            steps {
                sh 'docker compose run web bandit app.py || true'
            }
        }

        stage('Dependency Scan (Safety)') {
            steps {
                sh 'docker compose run web safety scan -r requirements.txt || true'
            }
        }

        stage('Image Scan (Trivy)') {
            steps {
                sh 'trivy image devsecops-lab-web  || true'
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
