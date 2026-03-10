pipeline {
    agent any

    environment {
        REGISTRY    = '10.1.1.176'
        IMAGE_REPO  = 'my-project/my-project'
        IMAGE_TAG   = "${BUILD_NUMBER}"
        FULL_IMAGE  = "${REGISTRY}/${IMAGE_REPO}:${IMAGE_TAG}"

        GITOPS_REPO   = 'https://github.com/park751115/my-project-gitops.git'
        GITOPS_BRANCH = 'main'
        GITOPS_FILE   = 'apps/dev/my-project/deployment.yaml'
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build -t ${FULL_IMAGE} .
                '''
            }
        }

        stage('Push Image to Harbor') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'harbor-creds',
                    usernameVariable: 'HARBOR_USER',
                    passwordVariable: 'HARBOR_PASS'
                )]) {
                    sh '''
                        echo "${HARBOR_PASS}" | docker login ${REGISTRY} -u "${HARBOR_USER}" --password-stdin
                        docker push ${FULL_IMAGE}
                        docker logout ${REGISTRY}
                    '''
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-gitops-push',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PAT'
                )]) {
                    sh '''
                        rm -rf /tmp/my-project-gitops
                        git clone -b ${GITOPS_BRANCH} https://${GIT_USER}:${GIT_PAT}@github.com/park751115/my-project-gitops.git /tmp/my-project-gitops

                        cd /tmp/my-project-gitops
                        git config user.name "park751115"
                        git config user.email "152490486+park751115@users.noreply.github.com"

                        sed -i 's|image: .*|image: '"${FULL_IMAGE}"'|' ${GITOPS_FILE}

                        git add ${GITOPS_FILE}
                        git commit -m "Update image to ${FULL_IMAGE}" || true
                        git push origin ${GITOPS_BRANCH}
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
        }
    }
}
