pipeline {
    agent any

    environment {
        CHECKOV_REPORT = "checkov-report.sarif"
        CHECKOV_TARGET_DIR = "${env.WORKSPACE}/terragoat"
    }

    stages {
        stage('Install Checkov') {
            steps {
                script {
                    try {
                        echo "Installing Checkov..."
                        sh '''
                        which python3 || echo "Python3 not found!"
                        python3 --version || echo "Python3 not found!"
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install checkov
                        checkov --version
                        '''
                    } catch (Exception e) {
                        echo "Error installing Checkov: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }

        stage('Clone Vulnerable Repo') {
            steps {
                script {
                    try {
                        echo "Cloning vulnerable repo (Terragoat)..."
                        sh """
                        git clone https://github.com/bridgecrewio/terragoat ${CHECKOV_TARGET_DIR}
                        """
                    } catch (Exception e) {
                        echo "Error cloning repo: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }

        stage('Run Checkov Scan') {
            steps {
                script {
                    try {
                        echo "Running Checkov scan on ${CHECKOV_TARGET_DIR}..."
                        sh """
                        . venv/bin/activate
                        checkov -d ${CHECKOV_TARGET_DIR} -o sarif > ${CHECKOV_REPORT}
                        """
                    } catch (Exception e) {
                        echo "Error running Checkov scan: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }

        stage('Display SARIF Report') {
            steps {
                sh """
                echo "=== Checkov SARIF Report ==="
                cat ${CHECKOV_REPORT}
                """
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.sarif', fingerprint: true
        }
    }
}