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
                junit 'target/surefire-reports/*.xml'
            }
        }
        stage('Build') {
            steps {
                sh './mvnw package'
            }
        }
    }
}
