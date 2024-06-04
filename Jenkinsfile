pipeline {
    agent any

    parameters {
        choice(name: 'environment', choices: ['main', 'dev'], description: 'deploying environmanet')
        choice(name: 'tag', choices: ['v1.0', 'v1.1'], description: 'tag')
    }
    
    environment {
	    registryNamespace = "orcsin"
        repositoryName = "nodemain"
        imageName2 = "${registryNamespace}/${repositoryName}"
        imageName3 = "${repositoryName}"
        //dockerImageName = ""
        registryDocker = "orcsin/lab3"
        registryCredential = 'docker_id'
        dockerImage = ''

        imageReference = ''
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
                    dockerImageName = ""
                    if (GIT_BRANCH == 'main') {
                        dockerImageName = "nodemain"
                    } else if (GIT_BRANCH == 'dev') {
                        dockerImageName = "nodedev"
                    }
                    //sh "echo ${dockerImageName}"
                    //sh "docker build -t node${params.environment}:${params.tag} ."
                    //sh "docker build -t ${imageName}:v1.0 ."

                    imageReference = "node${params.environment}:${params.tag}"
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
                    def run_containers = sh(returnStdout: true, script: 'docker container ps -q').replaceAll("\n", " ")
                    def all_containers = sh(returnStdout: true, script: 'docker container ps -aq').replaceAll("\n", " ")
                    if (run_containers){
                        sh "docker container kill ${run_containers}"
                    }
                    if (all_containers){
                        sh "docker container rm ${all_containers}"
                    }
                    //sh "docker stop $(docker ps -aq)"
                    //sh "docker container kill \$(docker ps -q)"
                    //sh "docker rm $(docker ps -aq)"
                    //sh "docker container rm \$(docker ps -a -q)"
                    sh "docker run -d --expose 3000 -p 3000:3000 node${params.environment}:${params.tag}" 
                    /*
                    def port = ""
                    if (env.BRANCH_NAME == 'main') {
                        port = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        port = '3001'
                    }
                    sh "docker run -d --expose ${port} -p ${port}:3000 ${imageName}"

	    		    */
                    docker.withRegistry('', 'docker_id') {
                        dockerImage.push(${params.tag})
                    }
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
