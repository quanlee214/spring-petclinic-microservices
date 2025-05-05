pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'quanle214'
        IMAGE_TAG = "${GIT_COMMIT.take(7)}"
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

        stage('Build and Package Services') {
            steps {
                script {
                    def services = [
                        [name: 'spring-petclinic-vets-service', port: 8081],
                        [name: 'spring-petclinic-customers-service', port: 8082]
                        // bạn có thể thêm service khác vào đây nếu cần
                    ]

                    for (svc in services) {
                        def jarModule = svc.name
                        def jarBaseName = jarModule.replace('spring-petclinic-', '')
                        def jarPath = sh(
                            script: "ls ${jarModule}/target/${jarModule}-*.jar | grep -v original | head -n 1",
                            returnStdout: true
                        ).trim()
                        def dockerJarPath = "docker/${jarBaseName}.jar"

                        echo "▶️ Building ${svc.name}..."

                        // Build ứng dụng
                        sh "./mvnw -pl ${jarModule} -am clean package -DskipTests"

                        // Copy jar vào thư mục docker
                        sh "cp ${jarPath} ${dockerJarPath}"

                        echo "🐳 Building Docker image for ${svc.name}..."
                        sh """
                            docker build \
                                --build-arg ARTIFACT_NAME=${jarBaseName} \
                                --build-arg EXPOSED_PORT=${svc.port} \
                                -f docker/Dockerfile \
                                -t ${DOCKERHUB_USERNAME}/${jarBaseName}:${IMAGE_TAG} \
                                docker/
                        """
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"

                        def services = ['vets-service', 'customers-service']
                        for (svc in services) {
                            def image = "${DOCKERHUB_USERNAME}/${svc}:${IMAGE_TAG}"
                            echo "📤 Pushing ${image}"
                            sh "docker push ${image}"
                        }
                    }
                }
            }
        }
    }
}
