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
		script {
			def myDockerImage = docker.build("${registry}:${env.BUILD_ID}")
		}
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying Example'
		script {
			docker.withRegistry('https://index.docker.io', 'docker_id'){
				// docker.image("${registry}:${env.BUILD_ID}").push("${env.BUILD_ID}")
				myDockerImage.push('latest')
			}
		}
            }
        }
    }
    environment {
	registry = 'orcsin/nodemain'
	// registryCredential = 'docker_id'
    }
}

