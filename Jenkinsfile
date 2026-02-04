pipeline {
    agent any

    environment {
        APP_NAME = "cicd-web-demo"
        STAGING_PORT = "8081"
        PROD_PORT = "8082"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Código descargado."
            }
        }

        stage('Lint / Validation') {
            steps {
                echo "Validando estructura mínima..."
                sh 'test -f Dockerfile'
                sh 'test -f docker-compose.yml'
                sh 'test -f app/index.html'
                sh 'test -x scripts/test.sh'
                echo "Validation OK"
            }
        }

        stage('Test') {
            steps {
                echo "Ejecutando pruebas..."
                sh './scripts/test.sh'
            }
        }

        stage('Build Image (Staging)') {
            steps {
                echo "Construyendo imagen para staging..."
                sh "docker build -t ${APP_NAME}:staging ."
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo "Desplegando en STAGING (port ${STAGING_PORT})..."
                sh 'docker compose up -d web-staging'
                echo "Staging updated. Verify at: http://IP-VM:8081"
            }
        }

        stage('Approval for Production') {
            steps {
                input message: 'Approve deployment to PRODUCTION?', ok: 'Yes, deploy'
            }
        }

        stage('Promote Image to Production') {
            steps {
                echo "Promoting image to production..."
                sh "docker tag ${APP_NAME}:staging ${APP_NAME}:production"
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "Deploying to PRODUCTION (port ${PROD_PORT})..."
                sh 'docker compose up -d web-production'
                echo "Production updated. Verify at: http://IP-VM:8082"
            }
        }
    }

    post {
        success {
            echo "CI/CD completed successfully."
        }

        failure {
            echo "CI/CD failed. Check build logs."
        }

        always {
            sh """
            docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}' || true
            """
        }
    }
}
