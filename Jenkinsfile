pipeline {
    agent any

    stages {    
        stage('Registering build artifact') {
            steps {
                echo 'Registering the metadata'
                registerBuildArtifactMetadata(
                    name: "test-artifact-demo",
                    version: "1.0.1",
                    type: "docker",
                    url: "http://non:1111",
                    digest: "6f637064707039346163663237383938",
                    label: "prod"
                )
            }
        }

        stage('Register Security Scan') {
            steps {
                script {
                    if (fileExists("anchore-findings.json")) {
                        echo "File exists, registering scan..."
                        registerSecurityScan(
                            artifacts: "anchore-findings.json",
                            format: "",
                            scanner: "Anchore",
                            archive: false
                        )
                    } else {
                        error "anchore-findings.json not found!"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        failure {
            echo 'Build or tests failed!'
        }
    }
}