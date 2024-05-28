pipeline {
    agent any
    environment {
	    registryNamespace = "orcsin"
        repositoryName = "nodemain"
        imageName = "${registryNamespace}/${repositoryName}"
        registryCredential = 'docker_id'
        imageReference = ''
        dockerImage = ''
        branchName = ''
        port = ''
    }
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
                sh 'chmod +x ./scripts/build.sh'
		        sh 'scripts/build.sh'
            }
        }
        stage('Test') {
            steps {
		        echo 'Testing'
                sh 'chmod +x ./scripts/test.sh'
		        sh 'scripts/test.sh'
            }
        }
        stage('Build docker image') {
            steps {
                echo 'Build docker image'
		        script {
                    imageReference = "${imageName}:${BUILD_NUMBER}"
                    dockerImage = docker.build imageReference
			    }
            }
        }
	    stage('Scan Docker Image for Vulnerabilities') {
		    steps {
		    	script {
			    	def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${imageReference}", returnStdout: true).trim()
                    echo "Vulnerability report:\n${vulnerabilities}"
			    }
		    }
	    }
        stage('Deploy') {
            steps {
                echo 'Deploying Example'
	    	    script {
	    		    docker.withRegistry('', 'docker_id') {
	    			    dockerImage.push('latest')
	    		    }
	    	    }
           }
        }
    }
    post ('Start deploy pipeline') {
        success {
            script {
                branchName = env.BRANCH_NAME
                if (branchName == 'dev') {
                    postJobName = 'Deploy_to_dev'
                } else if (branchName == 'main') {
                    postJobName = 'Deploy_to_main'
                }

            }
            sh 'echo $branchName'
            // build job: postJobName, parameters: [string(name: 'IMAGE_REFERENCE', value: imageReference)]
        }
    }
}

