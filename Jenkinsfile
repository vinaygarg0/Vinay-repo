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
                   
                sh "ssh -o StrictHostKeyChecking=no  ec2-user@54.191.167.253  sudo systemctl start docker ; sudo  docker  rm -f test1"
                sh "ssh ec2-user@54.191.167.253  sudo  docker  rm -f test1"
                sh "ssh ec2-user@54.191.167.253 sudo docker run -itd -p 8084:8080  --name test1  mdhack/myjava:${BUILD_NUMBER}"
}
           }
       }
       stage('Verifying The Code') {
           steps {
               retry(7) {
                    sh 'curl --silent http://54.191.167.253:8084/java-web-app/ | grep -i sehwag'
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
                   
                sh "ssh -o StrictHostKeyChecking=no  ec2-user@35.90.247.185  sudo kubectl delete deployment mayank"
                sh "ssh ec2-user@35.90.247.185  sudo kubectl delete svc my-service"
                sh "ssh ec2-user@35.90.247.185  sudo rm -rf  webappsvc*"
                sh "ssh ec2-user@35.90.247.185 sudo kubectl create deployment mayank --image  mdhack/myjava:${BUILD_NUMBER}"
                sh "ssh ec2-user@35.90.247.185 sudo wget https://raw.githubusercontent.com/mdhack0316/jenkinsdockerapp/main/webappsvc.yml"
                sh "ssh ec2-user@35.90.247.185 sudo kubectl create -f webappsvc.yml"
}
           }
       }
        
        
    }
      post {
         always {
             echo "You can always see me"
         }
         success {
              echo "I am running because the job ran successfully"
         }
         unstable {
              echo "Gear up ! The build is unstable. Try fix it"
         }
         failure {
             echo "OMG ! The build failed"
             mail bcc: '', body: 'Ltest', cc: '', from: '', replyTo: '', subject: 'job ete fail', to: 'mayank123modi@gmail.com'
         }
     }

}

