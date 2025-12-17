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
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps{
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('push Image'){
            agent{
                docker{
                    image 'docker:27-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh '''
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
                     args '''
                      -u root
                      -v /var/jenkins_home/.kube/jenkins:/root/.kube
                      -v /var/jenkins_home/.minikube:/root/.minikube
                      --entrypoint=''
                    '''
                }
            }
            steps{
                sh '''
                export KUBECONFIG=/root/.kube/config
                kubectl config current-context
                kubectl get nodes
                kubectl create namespace dev --dry-run=client -o yaml | kubectl apply -f -
                kubectl apply -f k8s/ -n $KUBE_NAMESPACE
                kubectl rollout status deployment/expenses-web -n $KUBE_NAMESPACE
                '''
            }
        }
    }
}