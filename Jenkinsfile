pipeline {
    agent { label 'slave-nodes' }

    environment {
        DOCKER_IMAGE = "19901418/my-jenkins-python-app-ci-cd:latest"
        GIT_REPO     = "https://github.com/1418-jatin/PRT-Sample-Practise.git"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main',
                    url: "${GIT_REPO}",
                    credentialsId: 'dockerhub-cred'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
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
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            withCredentials([sshUserPrivateKey(credentialsId: 'pem-key', keyFileVariable: 'PEM_KEY')]) {
                sh """
                    ssh -i $PEM_KEY -o StrictHostKeyChecking=no ubuntu@54.163.199.47 \\
                        'export KUBECONFIG=/root/.kube/config && \\
                         kubectl apply -f /root/PRT/deployment.yml && \\
                         kubectl apply -f /root/PRT/service.yml && \\
                         kubectl rollout status deployment/foodexpress-deployment'
                """
            }
        }
    }
}
