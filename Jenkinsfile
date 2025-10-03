pipeline {
	agent any

	environment {
		DOCKER_HOST = 'tcp://dind:2375'
	}
	options {
		buildDiscarder(logRotator(numToKeepStr: '10'))
		timeout(time: 30, unit: 'MINUTES')
	}
	
	stages {
		stage('Checkout Code') {
			steps {
				git url: 'https://github.com/YiZhang0111/aws-elastic-beanstalk-express-js-sample.git', branch: 'main'
			}
		}

		stage('Install Dependencies') {
			steps {
				sh 'npm install --legacy-peer-deps'
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
					sh 'docker build -t myapp:latest .'
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
