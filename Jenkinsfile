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
          -Dsonar.sourceEncoding=UTF-8 \
          -Dsonar.login=$SONAR_AUTH_TOKEN
    '''
           }
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
                    dir('Docker/sonarqube') {
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