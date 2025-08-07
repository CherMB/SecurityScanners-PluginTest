pipeline {
    agent any

    environment {
        CHECKOV_REPORT = "checkov-report.sarif"
        CHECKOV_TARGET_DIR = "${env.WORKSPACE}/terragoat"
    }

    stages {
        stage('Install Checkov') {
            steps {
                sh '''
                echo "Installing Checkov..."
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install checkov
                checkov --version
                '''
            }
        }

        stage('Clone Vulnerable Repo') {
            steps {
                sh '''
                echo "Cloning vulnerable repo (Terragoat)..."
                git clone https://github.com/bridgecrewio/terragoat ${CHECKOV_TARGET_DIR}
                '''
            }
        }

        stage('Run Checkov Scan') {
            steps {
                sh '''
                echo "Running Checkov scan on ${CHECKOV_TARGET_DIR}..."
                . venv/bin/activate
                checkov -d ${CHECKOV_TARGET_DIR} -o sarif > ${CHECKOV_REPORT}
                '''
            }
        }

        stage('Display SARIF Report') {
            steps {
                sh '''
                echo "=== Checkov SARIF Report ==="
                cat ${CHECKOV_REPORT}
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.sarif', fingerprint: true
        }
    }
}