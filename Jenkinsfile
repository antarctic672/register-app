pipeline {
	agent { label 'Jenkins-Agent' }
	tools {
		jdk 'Java21'
		maven 'Maven3'
	}
	environment {
		APP_NAME = "register-app-pipeline"
		RELEASE = "1.0.0"
		DOCKER_USER = "antarctic672"
		IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
		IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	}
	
	stages{
		stage("Cleanup Workspace"){
			steps {
				cleanWs()
			}
		}

		stage("Checkout from SCM"){
			steps {
				git branch: 'main', credentialsId: 'github', url: 'https://github.com/antarctic672/register-app'
			}
		}


		stage("Build Application"){
			steps {
				sh "mvn clean package"
			}
		}


		stage("Test Application"){
			steps {
				sh "mvn test"
			}
		}

		stage("SonarQube Analysis"){
			steps {
				script {
					withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
						sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar'
					}
				}
			}
		}


		stage("Quality Gate") {
			steps {
				script {
					timeout(time: 5, unit: 'MINUTES') {
           	 			waitForQualityGate abortPipeline: true
					}
				}
			}
		}


		stage("Build & Push Docker Image") {
			steps {
				script {
					docker.withRegistry('https://registry-1.docker.io', 'jenkins-dockerhub-token') {
						docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
						docker_image.push()
						docker_image.push('latest')
					}
				}
			}
		}



		stage("Trivy Scan") {
			steps {
				script {
					sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image antarctic672/register-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
				}
			}
		}



		stage("Cleanup Artifacts") {
			steps {
				script {
					sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
					sh "docker rmi ${IMAGE_NAME}:latest"
				}
			}
		}


		stage("Trigger CD Pipeline") {
			steps {
				script {
					sh "curl -v -k --user antarctic672:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-100-55-71-95.compute-1.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
				}
			}
		}
	}
}
