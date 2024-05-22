pipeline {
    agent any
 
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Checkout Example'
            }
        }
        stage('Build') {
            steps {
                echo 'Building Example'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing Example'
            }
        }
        stage('Build docker image') {
            steps {
                echo 'Build docker image'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying Example'
            }
        }
    }
}
