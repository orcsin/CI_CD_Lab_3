@Library("JenkinsTesLib@main") _

pipeline {
    agent any
 
    environment {
	    dockerNamespace = "orcsin"
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

        stage('Build docker image') {
            steps {
                echo 'Build docker image'
		        script {
                    def imageName = ''
                    if (env.BRANCH_NAME == 'main') {
                        imageName = "main"
                    } else if (env.BRANCH_NAME == 'dev') {
                        imageName = "dev"
                    }

                    imageReference = "${dockerNamespace}/node${imageName}:v1.0"
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
                DeployToMaster 'version'
	    	    script {
                    def run_containers = sh(returnStdout: true, script: 'docker container ps -q').replaceAll("\n", " ")
                    def all_containers = sh(returnStdout: true, script: 'docker container ps -aq').replaceAll("\n", " ")
                    def all_containers_pattern = sh(returnStdout: true, script: "docker ps -aq --filter ancestor=${imageReference}").replaceAll("\n", " ")
                    if (run_containers){
                        sh "docker container kill ${run_containers}"
                    }
                    if (all_containers){
                        sh "docker container rm ${all_containers_pattern} | true"
                    }              
                    
                    def port = ""
                    if (env.BRANCH_NAME == 'main') {
                        port = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        port = '3001'
                    }
                    def container = dockerImage.run("--expose ${port} -p ${port}:3000")
                    sleep 10
                    container.stop()
                }
	    	}
        }

        stage('Push') {
            steps {
                script {
                    sh "echo ${dockerImage}"
                    docker.withRegistry('', 'docker_id') {
                        docker.image(imageReference).push('latest')
                    }   
                }
            }
        }
    }
 
    post ('Clean docker image') {
        success {
            script {
                def branchName = env.BRANCH_NAME
                if (branchName == 'dev') {
                    postJobName = 'Deploy_to_dev'
                } else if (branchName == 'main') {
                    postJobName = 'Deploy_to_main'
                }
                build job: "${postJobName}"
            }
        }
    }
}
