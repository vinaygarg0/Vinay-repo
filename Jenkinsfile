pipeline {
    agent any

    environment {
        // Define environment variables
        IMAGE_NAME = 'vinay6may/vinayimgage'
        DEPLOYMENT_NAME_BLUE = 'vinayv1'
        DEPLOYMENT_NAME_GREEN = 'vinayv2'
        SERVICE_NAME = 'vinay-service'
        PORT = 80
        TARGET_PORT = 8080
    }

    stages {
        stage('Pulling The Code') {
            steps {
                git branch: 'main', url: 'https://github.com/vinaygarg0/Vinay-repo', credentialsId: 'GIT_CREDENTIALS'
            }
        }

        stage('Build Jar File Using Maven Tool') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Building Docker Image') {
            steps {
                sh 'sudo docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Pushing the Image Into Docker HUB') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'DOCKER_HUB_CREDENTIALS')]) {
                    sh 'sudo docker login -u vinay6may -p $DOCKER_HUB_CREDENTIALS'
                    sh 'sudo docker push ${IMAGE_NAME}:${BUILD_NUMBER}'
                }
            }
        }

        stage('Test Docker Container in DEV ENV') {
            steps {
                sh 'sudo docker rm -f test || true'
                sh 'sudo docker run -itd -p 8088:8080 --name test ${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }

        stage('Deploy in Production Environment') {
            steps {
                script {
                
                    // Create or update the green deployment
                    sh "kubectl create deployment ${DEPLOYMENT_NAME_GREEN} --image=${IMAGE_NAME}:${BUILD_NUMBER} "

                    // Update or create service to point to green deployment
                    sh '''
                    kubectl expose deployment ${DEPLOYMENT_NAME_GREEN} --port=${PORT} --target-port=${TARGET_PORT} --name=${SERVICE_NAME} --type=LoadBalancer || kubectl apply -f service-${SERVICE_NAME}.yml
                    '''
                    
                    }
                    
                    // Wait for the new deployment to become available
                    sh 'kubectl rollout status deployment/${DEPLOYMENT_NAME_GREEN}'
                    
                    // Optional: Validate the new deployment
                    sh 'curl --silent --fail http://<LOAD_BALANCER_IP>:${PORT}/health || exit 1'
                    
                    // Switch traffic from blue to green
                    echo 'Traffic switched to the green deployment.'
                }
            }
        }

        stage('Cleanup Blue Environment') {
            steps {
                script {
                    // Optionally delete the old blue deployment if required
                    sh "kubectl delete deployment ${DEPLOYMENT_NAME_BLUE}"
                    echo 'Old blue deployment cleaned up.'
                }
            }
        }
    }

    post {
        success {
            echo 'Blue-Green Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
            // Rollback logic (if necessary)
            sh "kubectl rollout undo deployment/${DEPLOYMENT_NAME_GREEN}"
        }
    }
