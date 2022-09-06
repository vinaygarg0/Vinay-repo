pipeline {
    agent any 
    stages {
        stage('gitcode Download') {
            steps {
                git branch: 'main', url: 'https://github.com/mdhack0316/jenkinsdockerapp'
            }
        }
        stage('Build Using Maven') {
            steps {
                sh 'mvn clean package'
            }    
        }
        stage('Building Own DockerImage') {
            steps {
                sh 'docker build -t mdhack/myjava:${BUILD_NUMBER} . '
            }
        }
       stage('Push Image to Docker HUB') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PWD', variable: 'DOCKER_HUB_PASS_CODE')]) {
    // some block
    sh 'docker login -u mdhack -p $DOCKER_HUB_PASS_CODE'
}
                sh 'docker push mdhack/myjava:${BUILD_NUMBER}'
            }
       }
       stage('Deploy in Dev Env') {
           steps { 
               sh 'docker rm -f test1'
               sh 'docker run -itd -p 8084:8080 --name test1 mdhack/myjava:${BUILD_NUMBER} '
           }
       }
       stage('Deploy App in QA/Test Env') {
           steps {
               sshagent(['QA_ENV_PASS']) {
                   
                sh "ssh -o StrictHostKeyChecking=no  ec2-user@54.186.147.248  sudo docker  rm -f test1"
                sh "ssh ec2-user@54.186.147.248 sudo docker run -itd -p 8084:8080  --name test1  mdhack/myjava:${BUILD_NUMBER}"
}
           }
       }
       
       stage('QA Team Test') {
           steps {
               input(message: "Release to Production? ")
           }
       }
      
       stage('Deploy App in PROD Env') {
           steps {
               sshagent(['QA_ENV_PASS']) {
                   
                sh "ssh -o StrictHostKeyChecking=no  ec2-user@52.37.113.238  sudo docker  rm -f test1"
                sh "ssh ec2-user@52.37.113.238 sudo docker run -itd -p 8084:8080  --name test1  mdhack/myjava:${BUILD_NUMBER}"
}
           }
       }
    }
}
