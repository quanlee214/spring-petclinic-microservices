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
                sh './mvnw clean verify'
            }
        }
        stage('Build') {
            steps {
                sh './mvnw package'
            }
        }
    }
    post {
        always {
            // ✅ Upload kết quả test
            junit '**/target/surefire-reports/*.xml'
            
            // ✅ Upload coverage (nếu đã cài JaCoCo plugin)
            jacoco execPattern: '**/target/jacoco.exec',
                   classPattern: '**/target/classes',
                   sourcePattern: '**/src/main/java',
                   inclusionPattern: '**/*.class'
        }
    }
}
