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
						docker_image = docker.build("${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}")
						docker_image.push()
						docker_image.push('latest')
					}
				}
			}
		}
	}
}
