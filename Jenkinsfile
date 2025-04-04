pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh './mvnw test'
            }
            post {
                always {
                    junit '*/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Coverage') {
            steps {
                sh './mvnw verify'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw package'
            }
        }
    }
}
