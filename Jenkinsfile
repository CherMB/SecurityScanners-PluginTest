pipeline {
    agent any

    environment {
        CHECKOV_REPORT = "checkov-report.sarif"
        CHECKOV_TARGET_DIR = "${env.WORKSPACE}/terragoat"
    }

    stages {
        stage('Install Pipenv & Checkov') {
            steps {
                script {
                    try {
                        echo "Installing Pipenv and Checkov..."
                        // Install pipenv without sudo
                        sh 'curl https://raw.githubusercontent.com/pypa/pipenv/master/get-pipenv.py | python3 -'
                        // Install checkov via pipenv
                        sh 'pipenv install checkov'
                        // Install checkov
                        sh 'pipenv run checkov --version'
                    } catch (Exception e) {
                        echo "Error installing Pipenv/Checkov: ${e.message}"
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
                        // Run checkov scan using pipenv
                        sh """
                        pipenv run checkov -d ${CHECKOV_TARGET_DIR} -o sarif > ${CHECKOV_REPORT}
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
                sh '''
                echo "=== Checkov SARIF Report ==="
                cat ${CHECKOV_REPORT}
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '${CHECKOV_REPORT}', fingerprint: true
        }
    }
}