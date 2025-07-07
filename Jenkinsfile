pipeline {
    agent any

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
                    withEnv(["PATH+SCANNER=${tool 'sonar-scanner'}/bin"]) {
                        sh '''
                            sonar-scanner \
                              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                              -Dsonar.sources=. \
                              -Dsonar.language=js \
                              -Dsonar.sourceEncoding=UTF-8
                        '''
                    }
                }
            }
        }

        stage('Esperar Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Desplegar en Docker') {
            steps {
                script {
                    def projectName = "wallipot"

                    // Nos aseguramos de estar en el directorio correcto del docker-compose.yml
                    dir('Docker') {
                        sh "docker-compose -p ${projectName} down -v --remove-orphans"
                        sh "docker-compose -p ${projectName} up -d --build"
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
            echo '✅ ¡Despliegue exitoso! Accede a http://localhost:8080'
        }
        failure {
            echo '❌ El pipeline falló.'
        }
    }
}
