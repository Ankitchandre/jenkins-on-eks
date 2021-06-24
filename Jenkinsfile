pipeline {
    agent {
        kubernetes{
            label 'jenkins-slave'
        }

    }
    environment{
        DOCKER_USERNAME = credentials('DOCKER_USERNAME')
        DOCKER_PASSWORD = credentials('DOCKER_PASSWORD')
    }
    stages {
        stage('docker login') {
            steps{
                sh(script: """
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                """, returnStdout: true)
            }
        }

        stage('git clone') {
            steps{
                sh(script: """
                    git clone https://github.com/Ankitchandre/jenkins-on-eks.git
                """, returnStdout: true)
            }
        }

        stage('docker build') {
            steps{
                sh script: '''
                #!/bin/bash
                cd $WORKSPACE/test-app/python
                docker build . --network host -t achandre/python:${BUILD_NUMBER}
                '''
            }
        }

        stage('docker push') {
            steps{
                sh(script: """
                    docker push achandre/python:${BUILD_NUMBER}
                """)
            }
        }

        stage('deploy') {
            steps{
                sh script: '''
                #!/bin/bash
                cd $WORKSPACE/jenkins-on-eks/
                #get kubectl for this demo
                curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
                chmod +x ./kubectl
                ./kubectl apply -f ./python-app/configmaps/configmap.yaml -n dev
                ./kubectl apply -f ./python-app/secrets/secret.yaml -n dev
                cat ./python-app/deployments/deployment.yaml | sed s/1.0.0/${BUILD_NUMBER}/g | ./kubectl apply -n dev -f -
                ./kubectl apply -f ./python-app/services/service.yaml -n dev
                '''
        }
    }
}
}
