pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "3.34.138.214"
        EC2_KEY  = "/var/jenkins_home/.ssh/id_rsa"

        APP_DIR  = "/home/ubuntu/app"
        APP_NAME = "jenkins_test-0.0.1-SNAPSHOT.jar"
        APP_PORT = "9090"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/gns0704/myapp.git', branch: 'main'
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

        stage('Deploy & Run on EC2') {
            steps {
                sh """
echo " Deploy start"


scp -i ${EC2_KEY} -o StrictHostKeyChecking=no \
build/libs/${APP_NAME} \
${EC2_USER}@${EC2_HOST}:${APP_DIR}/${APP_NAME}


ssh -i ${EC2_KEY} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
pkill -f ${APP_NAME} || true
echo "Starting new version"

nohup java -jar ${APP_DIR}/${APP_NAME} \\
  --server.address=0.0.0.0 \\
  --server.port=${APP_PORT} \\
  > ${APP_DIR}/app.log 2>&1 &
EOF
"""
            }
        }
    }

    post {
        success {
            echo " Deployment completed: http://${EC2_HOST}:${APP_PORT}"
        }
        failure {
            echo " Deployment failed"
        }
    }
}
