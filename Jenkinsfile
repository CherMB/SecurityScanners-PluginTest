pipeline {
    agent any

    environment {
        PYTHON_URL = "https://github.com/indygreg/python-build-standalone/releases/download/20240107/cpython-3.11.7+20240107-x86_64-unknown-linux-gnu-install_only.tar.gz"
        PYTHON_DIR = "${env.WORKSPACE}/python"
        VENV_DIR = "${env.WORKSPACE}/venv"
        CHECKOV_REPORT = "checkov-report.sarif.json"
        CHECKOV_TARGET_FILE = "${env.WORKSPACE}/minimain.tf"
        CHECKOV_DISABLE_GUIDE = "true"
    }

    stages {
        // Step 1: Download and set up prebuilt Python binary
        stage('Download Prebuilt Python') {
            steps {
                echo ":arrow_down: Downloading prebuilt Python binary..."
                sh '''
                    mkdir -p $PYTHON_DIR
                    cd $PYTHON_DIR
                    curl -L -o python.tar.gz $PYTHON_URL
                    tar -xzf python.tar.gz --strip-components=1
                    echo ":white_check_mark: Python extracted to: $PYTHON_DIR"
                '''
            }
        }

        // Step 2: Verify Python & Pip installation
        stage('Verify Python & Pip') {
            steps {
                sh '''
                    $PYTHON_DIR/bin/python3.11 --version
                    $PYTHON_DIR/bin/pip3.11 --version
                '''
            }
        }

        // Step 3: Create Virtual Environment for Pipenv
        stage('Create Virtual Environment') {
            steps {
                echo "🐍 Creating virtual environment if missing..."
                sh '''
                    if [ ! -d "$VENV_DIR" ]; then
                        $PYTHON_DIR/bin/python3.11 -m venv "$VENV_DIR"
                    else
                        echo "✅ Virtualenv already exists."
                    fi
                '''
            }
        }

        // Step 4: Install Pipenv if missing
        stage('Install Pipenv if Missing') {
            steps {
                echo "📦 Installing Pipenv if missing..."
                sh '''
                    source "$VENV_DIR/bin/activate"
                    if ! pip show pipenv > /dev/null 2>&1; then
                        pip install pipenv
                    else
                        echo "✅ Pipenv already installed."
                    fi
                '''
            }
        }

        // Step 5: Install Checkov via Pipenv
        stage('Install Checkov via Pipenv') {
            steps {
                echo "📦 Installing Checkov using Pipenv..."
                sh '''
                    source "$VENV_DIR/bin/activate"
                    pip install certifi
                    pipenv install checkov==3.2.460  # Ensure Checkov version is correct
                    echo "✅ Checkov and certifi installed."
                '''
            }
        }

        // Step 6: Checkov Version Debugging
        stage('Checkov Version') {
            steps {
                echo "🔍 Checking Checkov Version"
                sh '''
                    source "$VENV_DIR/bin/activate"
                    pipenv run checkov --version
                '''
            }
        }

        // Step 7: Run Checkov Scan with SARIF Output (ensure clean JSON)
        stage('Run Checkov Scan') {
            steps {
                echo "🚨 Running Checkov scan on a specific file (main.tf)..."
                sh '''
                    source "$VENV_DIR/bin/activate"
                    export SSL_CERT_FILE=$(python -m certifi)

                    # Run Checkov with SARIF output and redirect to the report
                    echo ":white_check_mark: Running Checkov with SARIF output..."
                    pipenv run checkov -f "$CHECKOV_TARGET_FILE" -o sarif --output "$CHECKOV_REPORT" > /dev/null 2>&1 || true
                    echo ":white_check_mark: SARIF report generated at $CHECKOV_REPORT"
                '''
            }
        }

        // Step 8: Display SARIF Report (first 20 lines for debugging)
        stage('Display SARIF Report') {
            steps {
                echo "📄 Displaying SARIF report (First 20 lines):"
                sh '''
                    echo "=== Checkov SARIF Report (First 20 lines) ==="
                    head -n 20 "$CHECKOV_REPORT"
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${CHECKOV_REPORT}", fingerprint: true
        }
    }
}
