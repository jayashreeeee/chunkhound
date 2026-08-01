pipeline {
    agent any

    environment {
        PATH = "$HOME/.local/bin:${env.PATH}"
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
                    echo "Current Directory:"
                    pwd

                    echo "Python Version:"
                    python3 --version

                    echo "UV Version:"
                    uv --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    uv sync
                '''
            }
        }

        stage('Build Package') {
            steps {
                sh '''
                    uv build
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    uv run chunkhound --help
                '''
            }
        }

        stage('List Artifacts') {
            steps {
                sh '''
                    ls -lh dist
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'dist/*', fingerprint: true
            echo 'Build completed successfully.'
        }

        failure {
            echo 'Build failed.'
        }

        always {
            cleanWs()
        }
    }
}
