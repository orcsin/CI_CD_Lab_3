pipeline {
    agent any
    environment {
	    registryNamespace = "orcsin"
        repositoryName = "nodemain"
        imageName = "${registryNamespace}/${repositoryName}"
        registryDocker = "orcsin/lab3"
        registryCredential = 'docker_id'
        imageReference = ''
        dockerImage = ''
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
        /*stage('change a port for main'){
            //steps {
                when {
                    branch 'main'
                }
                steps {
                    sh "echo 'EXPOSE 3000' >> Dockerfile"
                }
            //}
        */}
        stage('Build docker image') {
            steps {
                echo 'Build docker image'
		        script {
                    imageReference = "${imageName}:v1.0"
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
                
	    	    script {
	    		    docker.withRegistry('', 'docker_id') {
                        dockerImage.push('latest')
	    		    }
	    	    }
           }
        }
    }
    post ('Clean docker image') {
        success {
            script {
                sh 'docker rmi orcsin/nodemain:v1.0'
                def branchName = env.BRANCH_NAME
                if (branchName == 'dev') {
                    postJobName = 'Deploy_to_dev'
                } else if (branchName == 'main') {
                    postJobName = 'Deploy_to_main'
                }
                build job: postJobName, parameters: [string(name: 'IMAGE_REFERENCE', value: imageReference)]
            }
        }
    }
}
