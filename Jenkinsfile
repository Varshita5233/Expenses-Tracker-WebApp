pipeline{
    agent none
    environment{
        IMAGE_NAME = "jyotsna2181/expenses-tracker-webapp"
        IMAGE_TAG = "latest"
        KUBE_NAMESPACE = "dev"
    }

    stages{
        stage('Checkout'){
            agent any
            steps{
                checkout scm
            }
        }

        stage('Build JAR'){
            agent{
                docker{
                    image 'maven:3.9.9-eclipse-temurin-17'
                }
            }
            steps{
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image'){
            agent{
                docker{
                    image 'docker:27-cli'
                    args '''
                    -u root
                    -v /var/run/docker.sock:/var/run/docker.sock
                    -e DOCKER_CONFIG=/tmp/.docker
                    '''
                }
            }
            steps{
                sh '''
                mkdir -p /tmp/.docker
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('push Image'){
            agent{
                docker{
                    image 'docker:27-cli'
                    args '''
                    -u root
                    -v /var/run/docker.sock:/var/run/docker.sock
                    -e DOCKER_CONFIG=/tmp/.docker
                    '''
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh '''
                      mkdir -p /tmp/.docker
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage("Deploy to Kubernetes"){
            agent {
                docker {
                    image 'bitnami/kubectl:latest'
                    args '-v /var/lib/jenkins/.kube:/root/.kube'
                }
            }
            steps{
                sh '''
                kubectl apply -f k8s/namespace-dev.yaml
                kubectl apply -f k8s/
                kubectl rollout status deployment/expenses-web -n dev
                '''
            }
        }
    }
}