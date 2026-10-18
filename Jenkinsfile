pipeline {
    agent { label 'slave-nodes' }

    environment {
        DOCKER_IMAGE = "19901418/my-jenkins-python-app-ci-cd:latest"
        GIT_REPO = "https://github.com/1418-jatin/PRT-Sample-Practise.git"
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
            steps {
                // Use SSH to connect to devops-master and run kubectl
                withCredentials([sshUserPrivateKey(credentialsId: 'ubuntu-cred',
                                                  keyFileVariable: 'PEM_KEY')]) {
                    sh '''
                        scp -i $PEM_KEY -o StrictHostKeyChecking=no deployment.yaml service.yaml ubuntu@54.163.199.47:/home/ubuntu/
                        ssh -i $PEM_KEY -o StrictHostKeyChecking=no ubuntu@54.163.199.47 '
                        git clone https://github.com/1418-jatin/PRT-Sample-Practise.git ~/PRT-Sample-Practise || true
                            cd ~/PRT-Sample-Practise &&
                            kubectl apply -f deployment.yaml &&
                            kubectl apply -f service.yaml &&
                            kubectl rollout status deployment/myapp-deployment
                        '
                    '''
                }
            }
        }
    }
}
