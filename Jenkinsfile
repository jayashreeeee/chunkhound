pipeline {
    agent any

    tools {
        sonarQubeScanner 'SonarScanner'
    }

    environment {
        UV = "/opt/homebrew/bin/uv"
        PYTHON = "/opt/homebrew/bin/python3"
        SCANNER_HOME = tool 'SonarScanner'
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
                $PYTHON --version
                $UV --version
                '''
            }
        }

        stage('Create Virtual Environment') {
            steps {
                sh '''
                $UV venv --python $PYTHON
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                $UV sync
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                $UV build
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                $UV run chunkhound --help
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectKey=chunkhound \
                          -Dsonar.projectName=chunkhound \
                          -Dsonar.sources=. \
                          -Dsonar.python.version=3 \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.token=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
        success {
            archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
            cleanWs()
            echo 'Build and SonarQube Analysis Successful'
        }

        failure {
            echo 'Build Failed'
        }

        always {
            echo 'Pipeline Finished'
        }
    }
}
