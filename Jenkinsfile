pipeline {
    agent any

    environment {
        APP_NAME = "cicd-web-demo"
        STAGING_PORT = "8081"
        PROD_PORT = "8082"
    }

    options {
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Código descargado."
            }
        }

        stage('Validación / Lint') {
            steps {
                wrap([$class: 'TimestamperBuildWrapper']) {
                    echo "Validando estructura mínima..."
                    sh 'test -f Dockerfile'
                    sh 'test -f docker-compose.yml'
                    sh 'test -f app/index.html'
                    sh 'test -x scripts/test.sh'
                    echo "Validación OK"
                }
            }
        }

        stage('Pruebas') {
            steps {
                wrap([$class: 'TimestamperBuildWrapper']) {
                    echo "Ejecutando pruebas..."
                    sh './scripts/test.sh'
                }
            }
        }

        stage('Construir Imagen (Staging)') {
            steps {
                wrap([$class: 'TimestamperBuildWrapper']) {
                    echo "Construyendo imagen para staging..."
                    sh "docker build -t ${APP_NAME}:staging ."
                }
            }
        }

        stage('Desplegar a Staging') {
            steps {
                wrap([$class: 'TimestamperBuildWrapper']) {
                    echo "Desplegando en STAGING (puerto ${STAGING_PORT})..."
                    sh 'docker compose up -d web-staging'
                    echo "Staging actualizado. Verifica en: http://IP-VM:8081"
                }
            }
        }

        stage('Aprobación para Producción') {
            steps {
                input message: '¿Aprobar despliegue a PRODUCCIÓN?', ok: 'Sí, desplegar'
            }
        }

        stage('Promover Imagen a Producción') {
            steps {
                wrap([$class: 'TimestamperBuildWrapper']) {
                    echo "Promoviendo imagen a producción..."
                    sh "docker tag ${APP_NAME}:staging ${APP_NAME}:production"
                }
            }
        }

        stage('Desplegar a Producción') {
            steps {
                wrap([$class: 'TimestamperBuildWrapper']) {
                    echo "Desplegando en PRODUCCIÓN (puerto ${PROD_PORT})..."
                    sh 'docker compose up -d web-production'
                    echo "Producción actualizada. Verifica en: http://IP-VM:8082"
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD completado con éxito."
        }

        failure {
            echo "CI/CD falló. Revisar logs del build."
        }

        always {
            sh """
            docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}' || true
            """
        }
    }
}
