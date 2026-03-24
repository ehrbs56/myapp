pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "3.34.138.214"

        // 중요: 배포 디렉토리가 없으면 에러가 나므로 홈 디렉토리 활용 권장
        APP_DIR  = "/home/ubuntu/app"
        APP_NAME = "jenkins_test-0.0.1-SNAPSHOT.jar"
        APP_PORT = "9090"
    }

    stages {
        stage('Git Checkout') {
            steps {
                // 내 레포지토리 주소 확인됨
                git url: 'https://github.com/ehrbs56/myapp.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh '''
                # 1. Java 21 대신 Java 17을 사용하도록 강제 지정
                export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
                export PATH=$JAVA_HOME/bin:$PATH

                # 2. 권한 부여 및 빌드
                chmod +x ./gradlew
                ./gradlew clean bootJar -x test
                '''
            }
        }

        stage('Deploy & Run on EC2') {
            steps {
                // 3. /var/jenkins_home 경로 대신 안전한 sshagent 방식 사용
                sshagent(credentials: ['ec2-ssh']) {
                    sh """
                    echo "Deploy start to ${EC2_HOST}"

                    # 폴더가 없을 경우를 대비해 생성
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "mkdir -p ${APP_DIR}"

                    # 파일 전송 (sshagent 덕분에 -i 옵션 없이 전송 가능)
                    scp -o StrictHostKeyChecking=no build/libs/${APP_NAME} ${EC2_USER}@${EC2_HOST}:${APP_DIR}/${APP_NAME}

                    # 원격 실행
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                        # 기존 프로세스 종료 (포트 기준)
                        sudo fuser -k ${APP_PORT}/tcp || true

                        echo "Starting new version..."

                        # 백그라운드 실행
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
            echo "Deployment completed: http://${EC2_HOST}:${APP_PORT}"
        }
        failure {
            echo "Deployment failed"
        }
    }
}