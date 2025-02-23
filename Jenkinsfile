pipeline {
    agent none
    environment {
        IMAGE_NAME = 'belalelnady/simple-dynamic-app:latest'
        REMOTE_HOST = '192.168.1.111'
    }
    stages {
        stage('Checkout') {
            agent {label 'ssh-agent'}
            steps {
                // cant use `checkout scm` cuz it will be executed in  every stage 
                git url: 'https://github.com/belalelnady/dynamic-agent', branch: 'master'
                stash name: 'github-source', includes: '**'
            }
        }
        
        stage('Build Docker Image and push') {
            agent {label 'jenkins-agent-with-docker'}
            steps {
                unstash 'github-source'
                 withCredentials([usernamePassword(credentialsId: 'dockerHub', 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    script {

                         docker.build("${IMAGE_NAME}", './web-app')
                        sh """
                            docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}
                            docker push ${IMAGE_NAME}
                        """
                    }
                }
            }
        }
        
        stage('Deploy to Remote Server') {
            agent {label 'ssh-agent'}
            steps {
                sshagent (credentials: ['u-server']) {
                    sh """
                        mkdir -p ~/.ssh
                        #Remove the key if existed
                        ssh-keygen -R ${REMOTE_HOST} 2>/dev/null || true
                        ssh-keyscan -H ${REMOTE_HOST} >> ~/.ssh/known_hosts
                        chmod 600 ~/.ssh/known_hosts

                        ssh  belal@${REMOTE_HOST} '
                        docker container rm -f simple-app || true
                        docker image rm -f  ${IMAGE_NAME} || true
                        docker run -d -p 8770:80 --name simple-app ${IMAGE_NAME}'
                    """
                }
            }
        }
    }
    post {
        agent any
        success{
            sh 'echo sucess'
        }
        always {
            sh 'echo "build is finished"'
             cleanWs()
        }
    }
}
