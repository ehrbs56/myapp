pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_KEY  = "/var/jenkins_home/.ssh/id_rsa"

        APP_DIR  = "/home/ubuntu/app"
        APP_NAME = "jenkins_test-0.0.1-SNAPSHOT.jar"
        APP_PORT = "9090"

        //  순차 배포 대상 서버
        SERVERS  = "3.34.138.214 43.203.209.251"
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
chmod +x ./gradlew
./gradlew clean bootJar -x test
'''
            }
        }

        stage('Rolling Deploy') {
            steps {
                script {
                    def serverList = SERVERS.split()

                    for (int i = 0; i < serverList.size(); i++) {
                        def server = serverList[i]

                        echo " Deploying to ${server}"

                        // 1️ JAR 전송
                        sh """
scp -i ${EC2_KEY} -o StrictHostKeyChecking=no \
build/libs/${APP_NAME} \
${EC2_USER}@${server}:${APP_DIR}/${APP_NAME}
"""

                        // 2️⃣ 서버 재기동
                        sh """
ssh -i ${EC2_KEY} -o StrictHostKeyChecking=no ${EC2_USER}@${server} << EOF
pkill -f ${APP_NAME} || true
echo "Starting new version on ${server}"

nohup java -jar ${APP_DIR}/${APP_NAME} \\
  --server.address=0.0.0.0 \\
  --server.port=${APP_PORT} \\
  > ${APP_DIR}/app.log 2>&1 &
EOF
"""

                        // 3️⃣ 다음 서버로 넘어가기 전 대기 (무중단 핵심)
                        if (i < serverList.size() - 1) {
                            echo " Waiting before deploying next server..."
                            sleep(time: 30, unit: 'SECONDS')
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo " Rolling deployment completed successfully"
        }
        failure {
            echo " Deployment failed"
        }
    }
}
