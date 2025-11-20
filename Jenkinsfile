pipeline {
    agent any

    environment {
        APP_IMAGE = "myapp:latest"
        ZAP_IMAGE = "ghcr.io/zaproxy/zaproxy:stable"
        TARGET_URL = "http://65.1.113.27:5000"
        REPORT_NAME = "zap_report.html"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sandraimmaculate/tjfhm.git'
            }
        }

        stage('Clean Old Containers') {
            steps {
                sh 'docker rm -f myapp || true'
            }
        }

        stage('Build App Docker Image') {
            steps {
                sh 'docker build -t ${APP_IMAGE} .'
            }
        }

        stage('Run Application') {
            steps {
                sh 'docker run -d -p 5000:5000 --name myapp ${APP_IMAGE}'
                sh 'sleep 10'
            }
        }

        stage('ZAP Full Scan') {
            steps {
                echo 'Running ZAP security scan...'

                sh """
                    mkdir -p zap_reports
                    chmod 777 zap_reports

                    docker run --rm \
                        -v \$(pwd)/zap_reports:/zap/wrk \
                        ${ZAP_IMAGE} zap-full-scan.py \
                        -t ${TARGET_URL} \
                        -r ${REPORT_NAME} || true
                """
            }
        }

        stage('Publish ZAP Report') {
            steps {
                archiveArtifacts artifacts: "zap_reports/${REPORT_NAME}", allowEmptyArchive: false
            }
        }

    }
}
