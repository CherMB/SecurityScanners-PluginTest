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
                        // Install pipenv and Checkov using pipenv
                        sh '''
                        which pipenv || echo "Pipenv not found!"
                        pip install pipenv
                        pipenv install checkov
                        pipenv run checkov --version
                        '''
                    } catch (Exception e) {
                        echo "Error installing Pipenv/Checkov: ${e.message}"
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
            archiveArtifacts artifacts: '*.sarif', fingerprint: true
        }
    }
}