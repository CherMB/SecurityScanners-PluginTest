pipeline {
    agent any
    environment {
        PYTHON_URL = "https://github.com/indygreg/python-build-standalone/releases/download/20240107/cpython-3.11.7+20240107-x86_64-unknown-linux-gnu-install_only.tar.gz"
        PYTHON_DIR = "${env.WORKSPACE}/python"
        VENV_DIR = "${env.WORKSPACE}/venv"
        CHECKOV_REPORT = "checkov-report.sarif.json"
        CHECKOV_TARGET_FILE = "${env.WORKSPACE}/minimain.tf"
        JQ = "${WORKSPACE}/bin/jq"
        CHECKOV_DISABLE_GUIDE = "true"
        BC_API_KEY = ""
        PRISMA_API_URL = ""
    }
    stages {
        stage('Install JDK') {
            steps {
                sh '''
                    echo "Downloading JDK..."
                    curl -sLo openjdk.tar.gz https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.14%2B7/OpenJDK17U-jdk_x64_linux_hotspot_17.0.14_7.tar.gz
                    tar -xzf openjdk.tar.gz
                    rm -rf jdk17 && mv jdk-17* jdk17
                    mkdir -p ${WORKSPACE}/bin
                    if [ ! -f ${WORKSPACE}/bin/jq ]; then
                        echo "Downloading jq..."
                        curl -sLo ${WORKSPACE}/bin/jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
                        chmod +x ${WORKSPACE}/bin/jq
                    fi
                '''
            }
        }
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
        stage('Verify Python & Pip') {
            steps {
                sh '''
                $PYTHON_DIR/bin/python3.11 --version
                $PYTHON_DIR/bin/pip3.11 --version
                '''
            }
        }
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
        stage('Install Checkov via Pipenv') {
            steps {
                echo "📦 Installing Checkov using Pipenv..."
                sh '''
                source "$VENV_DIR/bin/activate"
                pip install certifi
                pipenv install checkov
                echo "✅ Checkov and certifi installed."
                '''
            }
        }
        stage('Run Checkov Scan') {
            steps {
                echo "🚨 Running Checkov scan on a specific file (main.tf)..."
                sh '''
                source "$VENV_DIR/bin/activate"
                export SSL_CERT_FILE=$(python -m certifi)
                CHECKOV_DISABLE_GUIDE=true
                pipenv run checkov -f "$CHECKOV_TARGET_FILE" -o sarif | ${JQ} -c '.' > "$CHECKOV_REPORT" || true
                '''
            }
        }
        stage('Display SARIF Report') {
            steps {
                echo "📄 Displaying SARIF report:"
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
