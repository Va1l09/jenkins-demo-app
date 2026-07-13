pipeline {
    agent any

    parameters {
        choice(
            name: 'ENTORNO',
            choices: ['staging', 'produccion'],
            description: 'Entorno de despliegue'
        )

        booleanParam(
            name: 'EJECUTAR_DEPLOY',
            defaultValue: false,
            description: '¿Desplegar al finalizar?'
        )
    }

    environment {
        APP_NAME = 'jenkins-demo-app'
        NODE_IMAGE = 'node:20-alpine'
        JENKINS_VOLUME = 'jenkins_home'
        PROJECT_PATH = '/var/jenkins_home/workspace/jenkins-demo-app_main'
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Clonando repositorio...'
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Instalar dependencias') {
            steps {
                echo 'Verificando Node.js y preparando el proyecto...'

                sh '''
                    docker run --rm \
                    -v ${JENKINS_VOLUME}:/var/jenkins_home \
                    -w ${PROJECT_PATH} \
                    ${NODE_IMAGE} node -v
                '''

                echo 'No hay dependencias externas en este demo.'
            }
        }

        stage('Test') {
            steps {
                echo 'Ejecutando pruebas automáticas...'

                sh '''
                    docker run --rm \
                    -v ${JENKINS_VOLUME}:/var/jenkins_home \
                    -w ${PROJECT_PATH} \
                    ${NODE_IMAGE} npm test
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Generando el paquete de la aplicación...'

                sh '''
                    docker run --rm \
                    -v ${JENKINS_VOLUME}:/var/jenkins_home \
                    -w ${PROJECT_PATH} \
                    ${NODE_IMAGE} npm run build
                '''

                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }

        stage('Deploy') {
            when {
                expression {
                    return params.EJECUTAR_DEPLOY
                }
            }

            steps {
                echo "Desplegando ${env.APP_NAME} en el entorno: ${params.ENTORNO}"
                echo 'Despliegue simulado completado.'
            }
        }
    }

    post {
        success {
            echo "Pipeline completado correctamente para ${env.APP_NAME}"
        }

        failure {
            echo 'El pipeline falló. Revisa los logs de la etapa correspondiente.'
        }

        always {
            echo "Build #${env.BUILD_NUMBER} finalizado."
        }
    }

    post {
    success {
        echo "Pipeline completado correctamente para ${env.APP_NAME}"
    }

    failure {
        echo 'El pipeline falló. Revisa los logs de la etapa correspondiente.'

        emailext(
            to: 'lknowsvl@gmail.com',
            subject: "Build fallido: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                El pipeline falló.

                Proyecto: ${env.JOB_NAME}
                Build: #${env.BUILD_NUMBER}
                Entorno: ${params.ENTORNO}

                Revisa la consola:
                ${env.BUILD_URL}
            """
        )
    }

    always {
        echo "Build #${env.BUILD_NUMBER} finalizado."
    }
}
}