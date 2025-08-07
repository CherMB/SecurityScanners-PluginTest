pipeline {
    agent any

    environment {
        CHECKOV_REPORT = "checkov-report.sarif"
        CHECKOV_TARGET_DIR = "${env.WORKSPACE}/terragoat"
    }

    stages {
        stage('Install Python, Pipenv & Checkov') {
            steps {
                script {
                    try {
                        echo "Installing Python, Pipenv, and Checkov..."

                        // Ensure Python and pip are installed
                        sh '''
                        if ! which python3; then
                            echo "Python3 not found, installing..."
                            sudo apt update && sudo apt install -y python3 python3-pip
                        else
                            echo "Python3 is already installed"
                        fi

                        if ! which pip; then
                            echo "Pip not found, installing..."
                            sudo apt install -y python3-pip
                        else
                            echo "Pip is already installed"
                        fi

                        if ! which pipenv; then
                            echo "Pipenv not found, installing..."
                            pip3 install pipenv
                        else
                            echo "Pipenv is already installed"
                        fi

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