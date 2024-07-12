pipeline{
	agent any
	tools{
		jdk "jdk17"
	}
	environment{
		SCANNER_HOME=tool "sonar-server"
		NEXUS_REPO_URL = 'http://192.168.121.129:8085/'
		NEXUS_REPO_CREDENTIALS_ID = 'nexus'
	}
	stages{
		stage("Clean Workspace"){
			steps{
				cleanWs()
			}
		}
		stage("Code Checkout"){
			steps{
				git branch: 'main', url: 'https://github.com/sreevarshitha1234/Zomato-Project.git'
			}
		}
		stage("Code Analysis by Sonar"){
			steps{
				withSonarQubeEnv('sonar-server'){
					sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato  -Dsonar.projectKey=zomato '''
				}
			}
		}
		stage("Code Quality Gates"){
			steps{
				script{
					timeout(time: 2, unit: 'MINUTES'){
					waitForQualityGate abortpipeline: false, credentialsId: 'sonar-server'
					}
				}
			}
		}
		stage("Install Dependencies"){
			steps{
				sh "npm install"
			}
		}
		stage("OWASP FS SCAN"){
			steps{
				dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
				dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
			}
		}
		stage("TRIVY FS SCAN"){
			steps{
				sh "trivy fs . > trivy.txt"
			}
		}
		stage("Docker Image Build"){
			steps{
				script{
				withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
					sh "docker build -t zomato . "
					sh "docker tag zomato 192.168.121.129:8085/zomato:latest"
				}
				}
			}
		}
		stage("Push Docker Image"){
		    steps{
				script{
					docker.withRegistry("https://${NEXUS_REPO_URL}", "${NEXUS_REPO_CREDENTIALS_ID}"){
					sh "docker push 192.168.121.129:8085/zomato:latest"
					}
				}
			}
		}
		stage("TRIVY is Image Scanning"){
			steps{
				sh "trivy image 192.168.121.129:8085/zomato:latest >trivy.txt"
			}
		}
		stage("Creating Docker Container"){
			steps{
				sh 'docker run -d --name zomato-app -h zomato -p 3000:3000 192.168.121.129:8085/zomato:latest'
			}
		}
	}
	post{
	failure{
	    emailext subject: "Build Failed: \${JOB_NAME} \${BUILD_NUMBER}",
	    body: "Hi Your Build Was Something Went Wrong, pls check once your build at \${BUILD_URL}",
	    to: "ravindra@cloudgensoft.com",
	    from: "ravindra.devops@gmail.com"
	}

	success{
	    emailext subject: "Build Success Ful: \${JOB_NAME} \${BUILD_NUMBER}",
	    body: "Hi Your Build Was Successful, pls check once your build for details at \${BUILD_URL}",
	    to: "ravindra@cloudgensoft.com",
	    from: "ravindra.devops@gmail.com"
	}
	}
}
