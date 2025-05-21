def globalServiceChanged = []
def commitId = ''

pipeline {
    agent any
    environment {
        DOCKERHUB_REPO = 'quanle214/' // Thay bằng repo DockerHub của bạn
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                    commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def changes = sh(script: 'git diff --name-only HEAD~1 HEAD', returnStdout: true).trim()

                    def serviceList = [
                        "spring-petclinic-admin-server",
                        "spring-petclinic-api-gateway",
                        "spring-petclinic-config-server",
                        "spring-petclinic-customers-service",
                        "spring-petclinic-discovery-server",
                        "spring-petclinic-vets-service",
                        "spring-petclinic-visits-service"
                    ]

                    for (svc in serviceList) {
                        if (changes.contains("${svc}/")) {
                            globalServiceChanged << svc
                        }
                    }

                    echo "Changed services: ${globalServiceChanged}"
                    echo "Commit ID: ${commitId}"
                }
            }
        }

        stage('Build & Push Docker Images') {
            when {
                expression { globalServiceChanged.size() > 0 }
            }
            steps {
                script {
                    def branches = [:]
                    globalServiceChanged.each { svc ->
                        branches[svc] = {
                            dir("${svc}") {
                                def imageTag = "${DOCKERHUB_REPO}${svc}:${commitId}"
                                echo "Building image: ${imageTag}"
                                sh '../mvnw clean install -P buildDocker -DskipTests'
                                sh "docker tag springcommunity/${svc}:latest ${imageTag}"

                                // 💥 Thêm docker login tại đây
                                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                                    sh """
                                        echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin
                                        echo "Pushing image: ${imageTag}"
                                        docker push ${imageTag}
                                    """
                                }
                            }
                        }
                    }

                    // Run in parallel
                    parallel branches
                }
            }
        }
    }

    post {
        success {
            echo "Build, push, and deploy completed for: ${globalServiceChanged}"
        }
    }
}
