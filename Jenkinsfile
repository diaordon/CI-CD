pipeline {
  agent any

  environment {
    WEBEX_TOKEN = credentials('webex-token')         // Jenkins secret text
    WEBEX_ROOM  = 'Y2lzY29zcGFyazovL3VybjpURUFNOnVzLXdlc3QtMl9yL1JPT00vMmRkZTQ2MTAtYTk0My0xMWYwLTkzZjEtMjU2MGEyMGI5ZDU1'      // <-- replace
  }

  options {
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install & Test') {
      steps {
        sh '''
          python3 --version
          python3 -m pip install --upgrade pip
          pip3 install -r requirements.txt
          mkdir -p test-results
          pytest -q --junitxml=test-results/pytest.xml
        '''
      }
    }
  }

  post {
    success {
      junit 'test-results/pytest.xml'
      sh '''
        curl -s https://webexapis.com/v1/messages \
          -H "Authorization: Bearer ${WEBEX_TOKEN}" \
          -H "Content-Type: application/json" \
          -d '{"roomId":"'"${WEBEX_ROOM}"'","text":"✅ Jenkins build ${JOB_NAME} #${BUILD_NUMBER} SUCCESS on ${GIT_BRANCH}"}' >/dev/null
      '''
    }
    failure {
      sh '''
        curl -s https://webexapis.com/v1/messages \
          -H "Authorization: Bearer ${WEBEX_TOKEN}" \
          -H "Content-Type: application/json" \
          -d '{"roomId":"'"${WEBEX_ROOM}"'","text":"❌ Jenkins build ${JOB_NAME} #${BUILD_NUMBER} FAILED. See: ${BUILD_URL}"}' >/dev/null
      '''
    }
    always {
      archiveArtifacts artifacts: 'test-results/pytest.xml', onlyIfSuccessful: false
    }
  }
}
