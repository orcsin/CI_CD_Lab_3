pipeline {
    agent any
 
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Checkout'
            }
        }
        stage('Build') {
            steps {
                sh 'scripts/build.sh'
                echo 'Building'
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
