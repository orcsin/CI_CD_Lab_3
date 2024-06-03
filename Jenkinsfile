pipeline {
    agent any
    environment {
	    registryNamespace = "orcsin"
        repositoryName = "nodemain"
        imageName2 = "${registryNamespace}/${repositoryName}"
        //imageName = ""
        registryDocker = "orcsin/lab3"
        registryCredential = 'docker_id'
        imageReference = ''
        dockerImage = ''
    }

    //triggers{ cron('H/1 * * * *') }

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
                    def imageName = ""
                    if (env.BRANCH_NAME == 'main') {
                        imageName = 'nodemain'
                    } else if (env.BRANCH_NAME == 'dev') {
                        imageName = 'nodedev'
                    }
                    //sh "docker build -t ${imageName}:v1.0 ."
                    imageReference = "${imageName2}:v1.0"
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
                    sh 'docker stop $(docker ps -aq)'
                    sh 'docker rm $(docker ps -aq)'

                    def port = ""
                    if (env.BRANCH_NAME == 'main') {
                        port = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        port = '3001'
                    }
                    sh "docker run -d --expose ${port} -p ${port}:3000 ${imageName}"

	    		    /*docker.withRegistry('', 'docker_id') {
                        dockerImage.push('latest')
	    		    */
                }
	    	}
        }
    }


    /*
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
    */
}
