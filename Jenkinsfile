pipeline {
    agent any

    environment {
        ENV_FILE = '.env'
    }

    stages {
        stage('Clonar Repositorio') {
            steps {
                script {
                    git 'https://github.com/YulianHernandez/bot-telegram-poli.git'
                }
            }
        }

        stage('Instalar Dependencias') {
            steps {
                script {
                    sh 'pip install -r app/requirements.txt'
                }
            }
        }

        stage('Configuración de la Base de Datos') {
            steps {
                script {
                    sh '''
                    mysql -h $DB_HOST -P $DB_PORT -u $DB_USER -p$DB_PASSWORD -e "CREATE DATABASE IF NOT EXISTS $DB_NAME;"
                    '''
                }
            }
        }

        stage('Ejecutar Bot') {
            steps {
                script {
                    sh 'python bot.py'
                }
            }
        }

        stage('Pruebas') {
            steps {
                script {
                    echo 'Ejecutando pruebas...'
                }
            }
        }

        stage('Desplegar a Producción') {
            steps {
                script {
                    echo 'Desplegando bot a producción...'
                    sh 'docker build -t telegram-bot .'
                    sh 'docker run -d -p 5000:5000 telegram-bot'
                }
            }
        }
    }

    post {
        success {
            echo 'El pipeline se ejecutó correctamente.'
        }
        failure {
            echo 'Hubo un error en el pipeline.'
        }
    }
}
