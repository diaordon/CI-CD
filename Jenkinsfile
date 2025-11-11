pipeline {
  agent any

  environment {
    WEBEX_TOKEN = credentials('webex-token')           // Jenkins Secret Text you created
    WEBEX_ROOM  = 'Y2lzY29zcGFyazovL3VybjpURUFNOnVzLXdlc3QtMl9yL1JPT00vNDA4ZjVkNjAtYmU5NC0xMWYwLTg3MGMtYmIzN2M0ZWE4MTVk'        // <-- your actual roomId
  }

  options { timestamps() }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install & Test') {
  steps {
    sh '''
      python3 --version
      python3 -m pip install --user -r requirements.txt
      mkdir -p test-results
      python3 -m pytest -q --junitxml=test-results/pytest.xml
    '''
  }
}


  post {
    success {
      junit 'test-results/pytest.xml'
      // Don’t fail the build if Webex API is down; send best-effort notice
      sh '''
        curl -sS -X POST https://webexapis.com/v1/messages \
          -H "Authorization: Bearer ${WEBEX_TOKEN}" \
          -H "Content-Type: application/json" \
          -d '{"roomId":"'"${WEBEX_ROOM}"'","text":"✅ Jenkins build ${JOB_NAME} #${BUILD_NUMBER} SUCCESS on ${GIT_BRANCH}"}' \
        || true
      '''
    }
    failure {
      sh '''
        curl -sS -X POST https://webexapis.com/v1/messages \
          -H "Authorization: Bearer ${WEBEX_TOKEN}" \
          -H "Content-Type: application/json" \
          -d '{"roomId":"'"${WEBEX_ROOM}"'","text":"❌ Jenkins build ${JOB_NAME} #${BUILD_NUMBER} FAILED. See: ${BUILD_URL}"}' \
        || true
      '''
    }
    always {
      archiveArtifacts artifacts: 'test-results/pytest.xml', onlyIfSuccessful: false
    }
  }
}

