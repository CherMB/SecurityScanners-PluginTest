pipeline {
    agent any

    environment {
        GO_VERSION = "1.21.5"
        CODEQL_BUNDLE_URL = "https://github.com/github/codeql-action/releases/latest/download/codeql-bundle-linux64.tar.gz"
        CODEQL_DIR = "${WORKSPACE}/codeql"
        GO_DIR = "${WORKSPACE}/go"
        GOPATH = "${WORKSPACE}/go-packages"
        GO_ROOT = "${GO_DIR}"
        PATH = "${GO_DIR}/bin:${PATH}"
        TEST_GO_PROJECT_DIR = "${WORKSPACE}/test-go-project"
        DB_NAME = "my-app-db-1"
        SARIF_OUTPUT = "results.sarif"
    }

    stages {
        stage('Install Go') {
            steps {
                echo "⬇️ Installing Go..."
                sh '''
                    set -e
                    GO_TEMP_DIR=$(mktemp -d)
                    curl -LO https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz
                    tar -xzf go${GO_VERSION}.linux-amd64.tar.gz -C "$GO_TEMP_DIR"
                    rm -rf "$GO_DIR"
                    mv "$GO_TEMP_DIR/go" "$GO_DIR"
                    echo "✅ Go installed at $GO_DIR"
                '''
            }
        }

        stage('Download and Extract CodeQL') {
            steps {
                echo "⬇️ Downloading CodeQL bundle..."
                sh '''
                    set -e
                    mkdir -p "$CODEQL_DIR"
                    curl -L "$CODEQL_BUNDLE_URL" -o codeql-bundle.tar.gz
                    tar -xzf codeql-bundle.tar.gz -C "$CODEQL_DIR" --strip-components=1
                    echo "✅ CodeQL installed to $CODEQL_DIR"
                '''
            }
        }

        stage('Create CodeQL Database') {
            steps {
                echo "📦 Creating CodeQL database from source..."
                sh '''
                    set -e
                    rm -rf "$DB_NAME"
                    export PATH="$GO_DIR/bin:$PATH"
                    export GOROOT="$GO_DIR"
                    export GOPATH="$GOPATH"
                    "$CODEQL_DIR/codeql" database create "$DB_NAME" \
                        --language=go \
                        --source-root="$TEST_GO_PROJECT_DIR"
                '''
            }
        }

        stage('Analyze Code with CodeQL') {
            steps {
                echo "🔍 Running CodeQL analysis..."
                sh '''
                    set -e
                    "$CODEQL_DIR/codeql" database analyze "$DB_NAME" \
                        codeql/go-queries \
                        --format=sarifv2.1.0 \
                        --output="$SARIF_OUTPUT"
                '''
            }
        }

        stage('Publish SARIF to Dashboard') {
            when {
                expression { fileExists("${SARIF_OUTPUT}") }
            }
            steps {
                echo "📤 SARIF file ready at ${SARIF_OUTPUT}"
                // Add your actual SARIF publishing logic here (e.g., upload to GitHub, etc.)
            }
        }
    }

    post {
        success {
            echo '✅ Build completed'
        }
        failure {
            echo '❌ Build failed'
        }
    }
}
