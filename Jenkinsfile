pipeline {
    agent any

    stages {
        stage('Verify') {
            steps {
                echo 'GitHub -> Jenkins 연결 확인'
                sh 'pwd'
                sh 'ls -la'
                sh 'git --version || true'
                sh 'java -version || true'
            }
        }
    }
}
