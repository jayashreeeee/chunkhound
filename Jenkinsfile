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

                    /opt/homebrew/bin/python3 --version
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {

                        sh '''
                        /opt/homebrew/bin/sonar-scanner \
                          -Dsonar.projectKey=chunkhound \
                          -Dsonar.projectName=chunkhound \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Verify Build Output') {
            steps {
                sh '''
                    echo "Current directory:"
                    pwd

                    echo "Workspace contents:"
                    ls -la

                    echo "Dist contents:"
                    ls -la dist
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'dist/*',
                             fingerprint: true,
                             allowEmptyArchive: true

            cleanWs()
        }

        success {
            echo 'Build Successful'
        }

        failure {
            echo 'Build Failed'
        }
    }
}
