pipeline {
    agent any

    environment {
        CODEQL_URL = "https://github.com/github/codeql-action/releases/latest/download/codeql-bundle-linux64.tar.gz"
        CODEQL_DIR = "${env.WORKSPACE}/codeql"
        SOURCE_DIR = "${env.WORKSPACE}/test-workflow-ninja" 
        DB_NAME = "my-app-db-1"
        SARIF_OUTPUT = "result1.sarif"
    }

    stages {
        stage('Download and Extract CodeQL') {
            steps {
                echo "⬇️ Downloading CodeQL bundle..."
                sh '''
                    mkdir -p "$CODEQL_DIR"
                    curl -L "$CODEQL_URL" -o codeql-bundle.tar.gz
                    tar -xzf codeql-bundle.tar.gz -C "$CODEQL_DIR" --strip-components=1
                    echo "✅ CodeQL installed to $CODEQL_DIR"
                '''
            }
        }

        stage('Create CodeQL Database') {
            steps {
                echo "📦 Creating CodeQL database from source..."
                sh '''
                    "$CODEQL_DIR/codeql" database create "$DB_NAME" \
                      --language=go \
                      --source-root="$SOURCE_DIR"
                '''
            }
        }

        stage('Analyze Code with CodeQL') {
            steps {
                echo "🔍 Running CodeQL analysis..."
                sh '''
                    "$CODEQL_DIR/codeql" database analyze "$DB_NAME" \
                      codeql/go-queries \
                      --format=sarifv2.1.0 \
                      --output="$SARIF_OUTPUT"
                '''
            }
        }

        stage('Display SARIF Report') {
            steps {
                echo "📄 SARIF Report Preview:"
                sh "head -n 20 $SARIF_OUTPUT || true"
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${SARIF_OUTPUT}", fingerprint: true
        }
    }
}
