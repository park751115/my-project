pipeline {
    agent any

    environment {
        APP_NAME        = 'my-project'
        HARBOR_REGISTRY = '10.1.1.176'
        HARBOR_PROJECT  = 'my-project'
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
        IMAGE_FULL      = "${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${APP_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh 'pwd'
                sh 'ls -la'
                sh 'docker --version'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${IMAGE_FULL} .
                '''
            }
        }

        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'harbor-creds',
                    usernameVariable: 'HARBOR_USER',
                    passwordVariable: 'HARBOR_PASS'
                )]) {
                    sh '''
                        echo "${HARBOR_PASS}" | docker login ${HARBOR_REGISTRY} -u "${HARBOR_USER}" --password-stdin
                        docker push ${IMAGE_FULL}
                        docker logout ${HARBOR_REGISTRY} || true
                    '''
                }
            }
        }
    }
}
