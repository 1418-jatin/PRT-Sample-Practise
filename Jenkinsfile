pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "19901418/my-jenkins-python-app-ci-cd:latest"
        EC2_HOST = "ec2-user@54.163.199.47"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/1418-jatin/PRT-Sample-Practise.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key',
                                                  keyFileVariable: 'EC2_KEY')]) {
                    sh """
                    ssh -i $EC2_KEY -o StrictHostKeyChecking=no ${EC2_HOST} '
                        docker pull ${DOCKER_IMAGE} &&
                        docker stop myapp || true &&
                        docker rm myapp || true &&
                        docker run -d --name myapp -p 8080:8080 ${DOCKER_IMAGE}
                    '
                    """
                }
            }
        }
    }
}
