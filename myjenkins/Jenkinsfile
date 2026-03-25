pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "3.34.138.214"
        APP_DIR  = "/home/ubuntu/app"
        APP_NAME = "jenkins_test-0.0.1-SNAPSHOT.jar"
        APP_PORT = "9090"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/ehrbs56/myapp.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh '''
                echo "Current Java Version:"
                java -version

                chmod +x ./gradlew

                # Gradle Toolchain 기능을 무시하고 현재 설치된 Java(21)로 빌드하도록 강제함
                ./gradlew clean bootJar -x test -Porg.gradle.java.installations.auto-detect=false -Porg.gradle.java.installations.auto-download=false
                '''
            }
        }

        stage('Deploy & Run on EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh']) {
                    sh """
                    echo "Deploying to ${EC2_HOST}..."
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "mkdir -p ${APP_DIR}"
                    scp -o StrictHostKeyChecking=no build/libs/${APP_NAME} ${EC2_USER}@${EC2_HOST}:${APP_DIR}/${APP_NAME}

                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                        sudo fuser -k ${APP_PORT}/tcp || true

                        # 실행할 때는 서버에 설치된 Java 17을 사용하게 됩니다.
                        nohup java -jar ${APP_DIR}/${APP_NAME} \
                          --server.port=${APP_PORT} \
                          > ${APP_DIR}/app.log 2>&1 &
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo " ✅ Deployment Completed: http://${EC2_HOST}:${APP_PORT}"
        }
        failure {
            echo " ❌ Deployment Failed"
        }
    }
}