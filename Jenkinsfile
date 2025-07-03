pipeline {
    agent any

    tools {
        sonarQubeScanner 'sonar-scanner'
    }

    environment {
        SONARQUBE_SERVER = 'sonarqube'
        SONAR_PROJECT_KEY = 'wallipot'
    }

    stages {
        stage('Clonar repositorio') {
            steps {
                cleanWs()
                git branch: 'main', url: 'https://github.com/JOANFALRO/walli-pot.git'
            }
        }

        stage('Análisis con SonarQube') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Espera Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Despliegue con Docker Compose') {
            steps {
                script {
                    def dockerProjectName = "wallipot-frontend"
                    dir('docker') {
                        sh "docker-compose -p ${dockerProjectName} down -v --remove-orphans"
                        sh "docker-compose -p ${dockerProjectName} up -d --build"
                        echo "Frontend desplegado en Docker. Verifica en: http://localhost:8080"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finalizado.'
        }
        success {
            echo '¡Despliegue exitoso!'
        }
        failure {
            echo 'Falló el pipeline.'
        }
    }
}