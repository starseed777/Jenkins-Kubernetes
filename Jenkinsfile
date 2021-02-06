pipeline {
    agent any

    environment {
    DOCKER_TAG = getDockerTag()
    registryCredential = "dockerhub"
    registry = "starseed777/nodeapp"
}

stages {
    stage('Build docker image') {
        steps{
            sh "docker build . -t starseed777/nodeapp:${DOCKER_TAG}"
        }
    }
    stage('push to dockerhub'){
        steps{
            script{
                docker.withRegistry('',registryCredential){
                sh "docker push starseed777/nodeapp:${DOCKER_TAG}"
            }
        }
    }
}

    stage('Deploy to kubernetes cluster'){
        steps{
            sh "chmod +x changeTag.sh"
            sh "./changeTag.sh ${DOCKER_TAG}"
            sshagent(['kops-machine']) {
            sh "scp -o StrictHostKeyChecking=no app-service.yml node-app-pod.yml ec2-user@18.212.169.39:/home/ec2-user"
            script {
                try {
                    sh "ssh ec2-user@3.85.169.2 kubectl apply -f . -n production"
                }catch(error) {
                    sh "ssh ec2-user@3.85.169.2 kubectl create -f . -n production"
                }
             }
            }
        }
    }
  }

}
def getDockerTag() {
    def tag = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag 
}
