pipeline {
    agent any

    environment {
        IMAGE_REPO = 'quanle214' 
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Tên nhánh cần build')
    }

    stages {
        stage('Clone source code') {
            steps {
                git branch: "${params.BRANCH_NAME}",
                    credentialsId: 'jenkins-docker',
                    url: 'https://github.com/quanlee214/spring-petclinic-microservices.git'
            }
        }

        stage('Lấy commit ID') {
            steps {
                script {
                    COMMIT_ID = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.IMAGE_TAG = COMMIT_ID
                    echo "✅ Commit ID: ${IMAGE_TAG}"
                }
            }
        }

        stage('Build & Push Docker images') {
            steps {
                script {
                    def services = ['vets-service', 'customers-service', 'visits-service'] 
                    for (svc in services) {
                        dir(svc) {
                            sh """
                                echo "🚧 Building ${svc} with tag ${IMAGE_TAG}..."
                                docker build -t ${IMAGE_REPO}/${svc}:${IMAGE_TAG} .
                                docker push ${IMAGE_REPO}/${svc}:${IMAGE_TAG}
                            """
                        }
                    }
                }
            }
        }
    }
}
