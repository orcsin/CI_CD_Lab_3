pipeline {
    agent any
    environment {
	    registryNamespace = "orcsin"
        repositoryName = "nodedev"
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
/*
	    stage('Scan Docker Image for Vulnerabilities') {
		    steps {
		    	script {
			    	def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${imageReference}", returnStdout: true).trim()
                    echo "Vulnerability report:\n${vulnerabilities}"
			    }
		    }
	    }
*/
        stage('Deploy') {
            steps {
                echo 'Deploying Example'
                sh 'docker images -a | grep ${imageName} | awk '{print $3}' | xargs docker rmi -f'
	    	    script {
	    		    docker.withRegistry('', 'docker_id') {
	    			    dockerImage.push('latest')
                        //dockerImage.push("${BUILD_NUMBER}")
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
                    port = 3000
                } else if (branchName == 'main') {
                    port = 3001
                }
                //echo "PORT = ${port}"
                //sh 'echo ${branchName}'
            } 
        }
            // build job: postJobName, parameters: [string(name: 'IMAGE_REFERENCE', value: imageReference)]
    }
}

