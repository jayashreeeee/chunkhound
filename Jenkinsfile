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

                    echo "Python:"
                    which python3 || true
                    python3 --version || true

                    echo "UV:"
                    which uv || true
                    uv --version

                    echo "Git:"
                    git --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    export PATH="/opt/homebrew/bin:$PATH"

                    uv sync
                '''
            }
        }

        stage('Build Package') {
            steps {
                sh '''
                    export PATH="/opt/homebrew/bin:$PATH"

                    uv build
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    export PATH="/opt/homebrew/bin:$PATH"

                    uv run chunkhound --help
                '''
            }
        }

        stage('Verify Build Artifacts') {
            steps {
                sh '''
                    echo "Artifacts generated:"
                    ls -lh dist
                '''
            }
        }
    }

    post {

        success {
            archiveArtifacts artifacts: 'dist/*', fingerprint: true
            echo 'Build Successful'
        }

        failure {
            echo 'Build Failed'
        }

        always {
            cleanWs()
        }
    }
}
