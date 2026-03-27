pipeline {
    agent any

    environment {
        // 우리 프라이빗 서버 정보
        EC2_USER = "ubuntu"
        EC2_HOST = "10.0.2.61"
        EC2_KEY  = "/var/jenkins_home/.ssh/id_rsa" // 젠킨스 컨테이너 안의 키 경로

        APP_DIR  = "/home/ubuntu/app"
        // 빌드 후 생성되는 실제 파일명 (아까 확인한 이름)
        APP_NAME = "jenkins_test-0.0.1-SNAPSHOT.jar"
        APP_PORT = "8080"
    }

    stages {
        stage('Git Checkout') {
            steps {
                // 내 깃허브 주소로 변경
                git url: 'https://github.com/ehrbs56/myapp1.git', branch: 'main'
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
                echo "🚀 Deploy start"

                # 1. JAR 파일 프라이빗 서버로 전송 (SCP 사용)
                scp -i ${EC2_KEY} -o StrictHostKeyChecking=no \
                build/libs/${APP_NAME} \
                ${EC2_USER}@${EC2_HOST}:${APP_DIR}/${APP_NAME}

                # 2. 서버에서 기존 프로세스 종료 후 재실행
                ssh -i ${EC2_KEY} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF

                # 기존에 실행 중인 자바 프로세스 종료
                pgrep -f ${APP_NAME} | xargs kill -15 || true

                echo "Starting new version..."

                # 백그라운드 실행
                nohup java -jar ${APP_DIR}/${APP_NAME} > ${APP_DIR}/app.log 2>&1 &

                echo "Deployment complete!"
EOF
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed!"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}