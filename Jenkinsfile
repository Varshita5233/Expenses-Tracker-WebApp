pipeline{
    agent any
    environment{
        IMAGE_NAME = "jyotsna2181/expenses-tracker-webapp"
        IMAGE_TAG = "latest"
        KUBE_NAMESPACE = "dev"
    }

    stages{
        stage('Checkout'){
            steps{
                checkout scm
            }
        }

        stage('Build JAR'){
            steps{
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image'){
            steps{
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('push Image'){
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