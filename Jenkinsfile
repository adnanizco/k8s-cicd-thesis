pipeline {
    agent any
    environment {
        IMAGE_A = "adnanizco/service-a:${BUILD_NUMBER}"
        IMAGE_B = "adnanizco/service-b:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS  = 'dockerhub-credentials'
        KUBECONFIG_CREDENTIAL = 'kubeconfig'
    }
    stages {
        stage('Checkout') { 
            steps { checkout scm } 
        }
        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE_A src/service-a'
                sh 'docker build -t $IMAGE_B src/service-b'
            }
        }
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${REGISTRY_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_A
                        docker push $IMAGE_B
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL}", variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl set image deployment/service-a service-a=$IMAGE_A --namespace=default
                        kubectl set image deployment/service-b service-b=$IMAGE_B --namespace=default
                    '''
                }
            }
        }
    }
    post { always { cleanWs() } }
}
