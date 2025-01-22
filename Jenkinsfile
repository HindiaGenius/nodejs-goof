pipeline {
    agent none
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DockerLogin')
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:lts-buster-slim'
                }
            }
            steps {
                sh 'npm install'
            }
        }
        stage('Build Docker Image and Push to Docker Registry') {
            agent {
                docker {
                    image 'docker:dind'
                    args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'docker build -t sryasetia/nodejsgoof:0.1 .'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push sryasetia/nodejsgoof:0.1'
            }
        }
        stage('Deploy Docker Image') {
            agent {
                docker {
                    image 'kroniak/ssh-client'
                    args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'DeploymentSSHKey', keyFileVariable: 'keyfile')]) {
                    sh '''
                        ssh -i ${keyfile} -o StrictHostKeyChecking=no $DEPLOY_USERNAME@$TARGET_IP \
                            "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                        ssh -i ${keyfile} -o StrictHostKeyChecking=no $DEPLOY_USERNAME@$TARGET_IP \
                            docker pull sryasetia/nodejsgoof:0.1
                        ssh -i ${keyfile} -o StrictHostKeyChecking=no $DEPLOY_USERNAME@$TARGET_IP \
                            docker rm --force mongodb
                        ssh -i ${keyfile} -o StrictHostKeyChecking=no $DEPLOY_USERNAME@$TARGET_IP \
                            docker run --detach --name mongodb -p 27017:27017 mongo:3
                        ssh -i ${keyfile} -o StrictHostKeyChecking=no $DEPLOY_USERNAME@$TARGET_IP \
                            docker rm --force nodejsgoof
                        ssh -i ${keyfile} -o StrictHostKeyChecking=no $DEPLOY_USERNAME@$TARGET_IP \
                            docker run -it --detach --name nodejsgoof --network host sryasetia/nodejsgoof:0.1
                    '''
                }
            }
        }
    }
}
