pipeline {
    agent any
    environment {
	    registryNamespace = "orcsin"
        repositoryName = "nodemain"
        imageName = "${registryNamespace}/${repositoryName}_${BRANCH_NAME}"
        registryCredential = 'docker_id'
        imageReference = ''
        dockerImage = ''

        registry = 'orcsin/nodemain'

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
                    imageReference = "${imageName}:${BUILD_NUMBER}"
                    dockerImage = docker.build imageReference
			        // def dockerImage = docker.build("${registry}:${env.BUILD_ID}")
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
	    			    dockerImage.push()
                        //docker.image("${registry}:${env.BUILD_ID}").push('latest')
	    		    }
	    	    }
           }
        }
    }
    post ('Start deploy pipeline') {
        success {
            script {
                def branchName = env.BRANCH_NAME
                if (branchName == 'dev') {
                    postJobName = 'Deploy_to_dev'
                } else if (branchName == 'main') {
                    postJobName = 'Deploy_to_main'
                }
            }
            build job: postJobName, parameters: [string(name: 'IMAGE_REFERENCE', value: imageReference)]
        }
    }
}

