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
                    // Danh sách dịch vụ: module name, jar tên rút gọn, cổng
                    def services = [
                        [module: 'spring-petclinic-vets-service', shortName: 'vets-service', port: 8081],
                        [module: 'spring-petclinic-customers-service', shortName: 'customers-service', port: 8082]
                        // Thêm dịch vụ khác nếu cần
                    ]

                    for (svc in services) {
                        def jarPathPattern = "${svc.module}/target/${svc.module}-*.jar"
                        def finalJar = sh(script: "ls ${jarPathPattern} | grep -v original | head -n 1", returnStdout: true).trim()
                        def dockerJarPath = "docker/${svc.shortName}.jar"

                        echo "▶️ Building ${svc.module}..."

                        // Build ứng dụng
                        sh "./mvnw -pl ${svc.module} -am clean package -DskipTests"

                        // Copy jar vào thư mục docker
                        sh "cp ${finalJar} ${dockerJarPath}"

                        echo "🐳 Building Docker image for ${svc.shortName}..."
                        sh """
                            docker build \
                                --build-arg ARTIFACT_NAME=${svc.shortName} \
                                --build-arg EXPOSED_PORT=${svc.port} \
                                -f docker/Dockerfile \
                                -t ${DOCKERHUB_USERNAME}/${svc.shortName}:${IMAGE_TAG} \
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
