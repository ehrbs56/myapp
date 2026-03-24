pipeline {
    agent any

    environment {
        // 배포 대상 EC2 정보
        EC2_USER = "ubuntu"
        EC2_HOST = "3.34.138.214"

        // 애플리케이션 설정
        APP_DIR  = "/home/ubuntu/app"
        APP_NAME = "jenkins_test-0.0.1-SNAPSHOT.jar"
        APP_PORT = "9090"
    }

    stages {
        stage('Git Checkout') {
            steps {
                // 내 깃허브 레포지토리에서 코드 가져오기
                git url: 'https://github.com/ehrbs56/myapp.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh '''
                echo "Checking Java version..."
                java -version

                # 만약 빌드 시 Java 17이 필요하다는 에러가 발생하면
                # 아래 주석(#)을 제거하고 경로를 수정하여 사용하세요.
                # export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
                # export PATH=$JAVA_HOME/bin:$PATH

                chmod +x ./gradlew
                ./gradlew clean bootJar -x test
                '''
            }
        }

        stage('Deploy & Run on EC2') {
            steps {
                // 젠킨스에 등록한 'ec2-ssh' 인증 정보를 사용
                sshagent(credentials: ['ec2-ssh']) {
                    sh """
                    echo "Starting Deployment to ${EC2_HOST}..."

                    # 1. 대상 서버에 애플리케이션 폴더 생성
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "mkdir -p ${APP_DIR}"

                    # 2. 빌드된 jar 파일을 대상 서버로 전송
                    scp -o StrictHostKeyChecking=no build/libs/${APP_NAME} ${EC2_USER}@${EC2_HOST}:${APP_DIR}/${APP_NAME}

                    # 3. 대상 서버에서 기존 프로세스 종료 및 새 버전 실행
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                        # 9090 포트를 사용하는 프로세스 종료
                        sudo fuser -k ${APP_PORT}/tcp || true

                        echo "Launching ${APP_NAME} on port ${APP_PORT}..."

                        # 백그라운드 실행 및 로그 기록
                        nohup java -jar ${APP_DIR}/${APP_NAME} \
                          --server.port=${APP_PORT} \
                          > ${APP_DIR}/app.log 2>&1 &

                        echo "Application started successfully."
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "-----------------------------------------------------------"
            echo " ✅ Deployment Completed!"
            echo " URL: http://${EC2_HOST}:${APP_PORT}"
            echo "-----------------------------------------------------------"
        }
        failure {
            echo "-----------------------------------------------------------"
            echo " ❌ Deployment Failed. Please check the Console Output."
            echo "-----------------------------------------------------------"
        }
    }
}