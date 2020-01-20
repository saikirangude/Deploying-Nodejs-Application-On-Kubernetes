pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t saikirangude-nodeapp:${DOCKER_TAG} "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'Docker-Hub-Credentials', variable: 'dockerHubPwd')]) {
                    sh "docker login -u saikirangude -p Gudesaikiran1!"
                    sh "docker tag saikirangude-nodeapp:${DOCKER_TAG} saikirangude/nodejs-in-centos:2.0"
                    sh "docker push saikirangude/nodejs-in-centos:2.0"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['K8S-Mumbai-Region-Test-Instance-1-Credentials']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ec2-user@13.232.113.17:/home/ec2-user/"
                    script{
                        try{
                            sh "ssh ec2-user@13.232.113.17 kubectl apply -f ."
                        }catch(error){
                            sh "ssh ec2-user@13.232.113.17 kubectl create -f ."
                        }
                    }
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
