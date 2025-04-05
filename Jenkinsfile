pipeline {
    agent any
    environment {
        CHANGED_DIRS = ''
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Detect Changes') {
            steps {
                script {
                    def changes = sh(
                        script: "git diff --name-only origin/main...HEAD | cut -d/ -f1,2 | sort -u",
                        returnStdout: true
                    ).trim()
                    CHANGED_DIRS = changes.split("\n")
                    echo "📂 Thư mục thay đổi: ${CHANGED_DIRS}"
                }
            }
        }

        stage('Test & Build Vets Service') {
            when {
                expression { return CHANGED_DIRS.any { it.contains('spring-petclinic-vets-service') } }
            }
            steps {
                dir('spring-petclinic-vets-service') {
                    sh '../../mvnw clean verify'
                    sh '../../mvnw package'
                }
            }
        }

        stage('Test & Build Customers Service') {
            when {
                expression { return CHANGED_DIRS.any { it.contains('spring-petclinic-customers-service') } }
            }
            steps {
                dir('spring-petclinic-customers-service') {
                    sh '../../mvnw clean verify'
                    sh '../../mvnw package'
                }
            }
        }

        stage('Test & Build Visits Service') {
            when {
                expression { return CHANGED_DIRS.any { it.contains('spring-petclinic-visits-service') } }
            }
            steps {
                dir('spring-petclinic-visits-service') {
                    sh '../../mvnw clean verify'
                    sh '../../mvnw package'
                }
            }
        }

        stage('Test & Build Admin Server') {
            when {
                expression { return CHANGED_DIRS.any { it.contains('spring-petclinic-admin-server') } }
            }
            steps {
                dir('spring-petclinic-admin-server') {
                    sh '../../mvnw clean verify'
                    sh '../../mvnw package'
                }
            }
        }

        stage('Test & Build API Gateway') {
            when {
                expression { return CHANGED_DIRS.any { it.contains('spring-petclinic-api-gateway') } }
            }
            steps {
                dir('spring-petclinic-api-gateway') {
                    sh '../../mvnw clean verify'
                    sh '../../mvnw package'
                }
            }
        }

        // 👉 Bạn có thể thêm các service khác nếu cần
    }

    post {
        always {
            junit '**/target/surefire-reports/*.xml'
            jacoco execPattern: '**/target/jacoco.exec',
                   classPattern: '**/target/classes',
                   sourcePattern: '**/src/main/java',
                   inclusionPattern: '**/*.class'
        }
    }
}
