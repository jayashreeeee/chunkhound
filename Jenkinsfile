pipeline {
    agent any

    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:${env.PATH}"
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
                    echo "========== Environment =========="
                    pwd

                    echo "PATH=$PATH"

                    echo "Python Version:"
                    which python3
                    python3 --version

                    echo "UV Version:"
                    which uv || true
                    /opt/homebrew/bin/uv --version

                    echo "Git Version:"
                    git --version
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

        stage('Build Package') {
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

        stage('Verify Artifacts') {
            steps {
                sh '''
                    echo "Contents of dist directory:"
                    ls -lh dist
                '''
            }
        }
    }

    post {

        success {
            script {
                if (fileExists('dist')) {
                    archiveArtifacts artifacts: 'dist/*', fingerprint: true
                } else {
                    echo "dist directory not found. Skipping artifact archive."
                }
            }
            echo "Build completed successfully."
        }

        failure {
            echo "Build failed. Check the stage logs above for the root cause."
        }

        always {
            cleanWs()
        }
    }
}
