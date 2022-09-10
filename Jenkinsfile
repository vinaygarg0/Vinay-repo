pipeline {
    agent any

    stages {
        stage('Pulling The Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mdhack0316/jenkinsdockerapp'
            }
        }
        stage('Build Jar File Using Maven Tool') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Building Docker Image') {
            steps {
                sh 'sudo docker build -t mdhack/myocp:${BUILD_NUMBER} .'
            }
        }
        stage('Pusing the Image Into Docker HUB') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWD', variable: 'DOCKERPASSWORD')]) {
    // some block
                 sh 'sudo docker login -u mdhack -p  $DOCKERPASSWORD'
                 sh 'sudo docker push mdhack/myocp:${BUILD_NUMBER}'
}
               
            }
        }
        stage('Test Dokcer Container IN DEV ENV ') {
            steps {
                sh 'sudo docker rm -f test'
                sh 'sudo docker run -itd -p 8088:8080 --name test mdhack/myocp:${BUILD_NUMBER}'
            }
        }
        stage('Testing In Test Env') {
            steps {
                sshagent(['TESTENV']) {
    // some block
                sh 'ssh -o StrictHostKeyChecking=no ec2-user@43.205.96.46  sudo docker rm -f  test1'
                sh 'ssh ec2-user@43.205.96.46  sudo docker run -itd -p 4545:8080 --name test1 mdhack/myocp:${BUILD_NUMBER}'
}
            }
        }
        stage('Verifying Our Webssite in TEST ENV') {
            steps {
                retry(7)  {
                    sh 'curl -s 43.205.96.46:4545/java-web-app/ | grep -i sehwag'
}
            }
        }
        stage('Asking For Production Release?') {
            steps {
                input 'Release To Production? '
            }
        }
        stage('Deploy in Production Env Grade Kubernetes Cluster') {
            steps {
                sshagent(['PRODENV']) {
                sh 'ssh -o StrictHostKeyChecking=no  ubuntu@3.17.93.71 kubectl delete all --all'
                sh 'ssh ubuntu@3.17.93.71 kubectl create deployment mayank --image mdhack/myocp:${BUILD_NUMBER}'
                sh 'ssh ubuntu@3.17.93.71  wget https://raw.githubusercontent.com/mdhack0316/jenkinsdockerapp/main/webappsvc.yml '
                sh 'ssh ubuntu@3.17.93.71 kubectl create -f webappsvc.yml'
}
            }
        }
    }  
}
