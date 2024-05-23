pipeline {
    agent any
 
    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout'
		checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Building'
		sh 'scripts/build.sh'
            }
        }
        stage('Test') {
            steps {
		echo 'Testing'
		sh 'scripts/test.sh'
            }
        }
        stage('Build docker image') {
            steps {
                echo 'Build docker image'
		sh 'docker build -t orcsin/nodemain:v1.0 .'		
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying Example'
            }
        }
    }
}
