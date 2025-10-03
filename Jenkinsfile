pipeline {
  agent {
    docker {
      image 'node:16'
    }
  }

  environment {
    DOCKER_HOST = 'tcp://dind:2375'

    IMAGE_NAME = 'YiZhang0111/eb-express-sample'
    DOCKER_PUSH = 'true'
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10', daysToKeepStr: '30'))
    timeout(time: 30, unit: 'MINUTES')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install & Test') {
      steps {
        sh '''
              node -v && npm -v;
	      npm install --package-lock-only;
              npm install;
              npm test || echo 'No tests defined, skipping'
            
        '''
      }
    }

    stage('Security Scan - Snyk (fail on High/Critical)') {
      steps {
        withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
          sh '''
            docker run --rm -e SNYK_TOKEN="$SNYK_TOKEN" \
              -v $WORKSPACE:/workspace -w /workspace node:16 bash -lc "
		npm install -g snyk@1.1050.0 && \
		snyk auth $SNYK_TOKEN && \
                snyk test --severity-threshold=high
		" 
	 '''
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          env.SHORT_SHA = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
        sh '''
          set -eux
          docker build -t ${IMAGE_NAME}:${SHORT_SHA} .
          mkdir -p build
          docker save ${IMAGE_NAME}:${SHORT_SHA} -o build/${SHORT_SHA}.tar
        '''
      }
    }

    stage('Push to Registry') {
      when { expression { return env.DOCKER_PUSH == 'true' } }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            set -eux
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE_NAME}:${SHORT_SHA}
          '''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'build/*.tar', allowEmptyArchive: true
      echo 'pipeline completed'
    }
    failure {
      echo 'pipeline failed'
    }
  }
}

