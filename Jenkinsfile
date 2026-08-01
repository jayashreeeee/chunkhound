pipeline {
    agent any

    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:${env.PATH}"
        UV_PYTHON = "/opt/homebrew/bin/python3"
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "===== Environment ====="

                    echo "PATH=$PATH"

                    which python3
                    python3 --version

                    echo "Homebrew Python"
                    /opt/homebrew/bin/python3 --version

                    echo "UV"
                    /opt/homebrew/bin/uv --version
                '''
            }
        }

        stage('Create Virtual Environment') {
            steps {
                sh '''
                    /opt/homebrew/bin/uv venv --python /opt/homebrew/bin/python3
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    /opt/homebrew/bin/uv sync
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    /opt/homebrew/bin/uv build
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    /opt/homebrew/bin/uv run chunkhound --help
                '''
            }
        }

        stage('Archive') {
            steps {
                sh 'ls -lh dist'
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'dist/*', fingerprint: true
        }

        always {
            cleanWs()
        }
    }
}
