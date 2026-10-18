pipeline {
    agent { label 'slave-nodes' }

    environment {
        DOCKER_REPO  = "19901418/my-jenkins-python-app-ci-cd"
        DOCKER_IMAGE = "19901418/my-jenkins-python-app-ci-cd:${BUILD_NUMBER}"
        MASTER_IP    = "54.163.199.47"
    }

    stages {
        // Jenkins already checks out the repo automatically (Declarative: Checkout SCM),
        // so no extra checkout stage is needed.

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} -t ${DOCKER_REPO}:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}
                        docker push ${DOCKER_REPO}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ubuntu-cred',
                                                  keyFileVariable: 'PEM_KEY')]) {
                    sh '''
                        SSH_OPTS="-i $PEM_KEY -o StrictHostKeyChecking=no"

                        # copy manifests from this checkout to the master
                        ssh $SSH_OPTS ubuntu@${MASTER_IP} "mkdir -p ~/PRT"
                        scp $SSH_OPTS deployment.yml service.yml ubuntu@${MASTER_IP}:~/PRT/

                        # set this build's image tag, apply, and wait for rollout
                        ssh $SSH_OPTS ubuntu@${MASTER_IP} "
                            sed -i 's|image: .*|image: ${DOCKER_IMAGE}|' ~/PRT/deployment.yml &&
                            kubectl apply -f ~/PRT/deployment.yml &&
                            kubectl apply -f ~/PRT/service.yml &&
                            kubectl rollout status deployment/foodexpress-deployment --timeout=120s
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
