pipeline {
    agent any

    stages {
        stage('Pulling The Code') {
            steps {
                git branch: 'main', url: 'https://github.com/vinaygarg0/jenkinsdockerapp'
            }
        }
        stage('Build Jar File Using Maven Tool') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Building Docker Image') {
            steps {
                sh 'sudo docker build -t vinay6may/vinayimg:${BUILD_NUMBER} .'
            }
        }
        stage('Pusing the Image Into Docker HUB') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'DOCKER_HUB_CREDENTIALS')]) {
    // some block
                 sh 'sudo docker login -u vinay6may -p  $DOCKER_HUB_CREDENTIALS'
                 sh 'sudo docker push vinay6may/vinayimg:${BUILD_NUMBER}'
}
               
            }
        }
        stage('Test Dokcer Container IN DEV ENV ') {
            steps {
                sh 'sudo docker rm -f test'
                sh 'sudo docker run -itd -p 8088:8080 --name test vinay6may/vinayimg:${BUILD_NUMBER}'
            }
        }
        
        stage('Deploy in Production Env Grade Kubernetes Cluster') {
            steps {
                sshagent(['PRODENV']) {
                sh 'kubectl create deployment vinay --image vinay6may/vinayimg:${BUILD_NUMBER}'
                sh 'wget https://raw.githubusercontent.com/vinaygarg0/jenkinsdockerapp/main/webappsvc.yml '
                sh 'kubectl create -f webappsvc.yml'
}
            }
        }
    }  
}
