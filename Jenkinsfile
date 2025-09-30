pipeline {
	agent {
		docker {
			image 'node:16'
			args '-u root:root'
		}	
	}

	environment {
		DOCKER_HOST = 'tcp://dind:2376'
		DOCKER_IMAGE = "YiZhang0111/nodejs-app:latest"
	}
	options {
		buildDiscarder(logRotator(numToKeepStr: '10'))
		timeout(time: 30, unit: 'MINUTES')
	}
	
	stages {
		stage('Install Dependencies') {
			steps {
				sh 'npm install --save'
			}
		}

		stage('Run Tests') {
			steps {
				sh 'npm test'
			}
		}

		stage('Security Scan') {
			steps {
				script {
					sh 'npm audit --audit-level high'
				}
			}
		}

		stage('Build Docker Image') {
			steps {
				script {
					sh 'docker build -t ${env.DOCKER_IMAGE} .'
				}
			}
		}

		stage('Push Docker Image') {
			steps {
				script {
					echo "image pushing skiped, needs docker hub credential"
				}
			}
		}
	}
	
	post {
		always {
			echo 'pipeline completed'
		}
		success {
			echo 'pipeline success'
		}
		failure {
			echo ' pipeline failed'
		}
	}
}
