pipeline {
    agent any
    tools{
        maven 'jenkins-maven'
    }
    stages{
        stage('Build Maven'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/TheSuryaTeja/shopping-cart.git']]])
                sh 'mvn clean install'
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    sh 'docker build -t iamsurya/shopping-cart-app -f Dockerfile.app .'
                    sh 'docker build -t iamsurya/shopping-cart-mysql -f Dockerfile.mysql .'
                }
            }
        }
        stage('Push image to Hub'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhub-pwd')]) {
                   sh 'docker login -u iamsurya -p ${dockerhubpwd}'

}
                   sh 'docker push iamsurya/shopping-cart-app'
                   sh 'docker push iamsurya/shopping-cart-mysql'

                   
                }
            }
        }

    }
}
