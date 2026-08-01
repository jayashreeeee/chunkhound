pipeline {
    agent any

    environment {
        ...
    }

    options {
        ...
    }

    stages {

        stage('Stage 1') {
            steps {
                ...
            }
        }

        stage('Stage 2') {
            steps {
                ...
            }
        }

        stage('Verify Build Output') {
            steps {
                sh '''
                    pwd
                    ls -la
                    ls -la dist || true
                '''
            }
        }
    }   

    post {

        success {
            echo 'Build Successful'
            archiveArtifacts artifacts: 'dist/*', fingerprint: true
        }

        failure {
            echo 'Build Failed'
        }

        always {
            cleanWs()
        }
    }  

}   
