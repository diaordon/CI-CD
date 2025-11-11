pipeline {
  agent any

  environment {
    WEBEX_TOKEN = credentials('webex-token')   // Jenkins secret text id
    WEBEX_ROOM  = 'Y2lzY29zcGFyazovL3VybjpURUFNOnVzLXdlc3QtMl9yL1JPT00vNDA4ZjVkNjAtYmU5NC0xMWYwLTg3MGMtYmIzN2M0ZWE4MTVk'      // your Webex roomId
  }

  options { timestamps() }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install & Test') {
      steps {
        sh '''
          set -euxo pipefail
          python3 --version
          # create isolated env so pip works in Debian-based Jenkins image
          python3 -m venv .venv
          . .venv/bin/activate
          pip install -U pip
          pip install -r requirements.txt
          mkdir -p test-results
          .venv/bin/pytest -q --junitxml=test-results/pytest.xml
        '''
      }
      post {
        always {
          junit 'test-results/pytest.xml'
          archiveArtifacts artifacts: 'test-results/pytest.xml', onlyIfSuccessful: false
        }
      }
    }
  }

  post {
    success {
      sh """
        curl -sX POST https://webexapis.com/v1/messages \
          -H 'Authorization: Bearer ${WEBEX_TOKEN}' \
          -H 'Content-Type: application/json' \
          -d '{\"roomId\":\"${WEBEX_ROOM}\",\"text\":\"✅ Jenkins build ${JOB_NAME} #${BUILD_NUMBER} SUCCESS on ${GIT_BRANCH}\"}'
      """
    }
    failure {
      sh """
        curl -sX POST https://webexapis.com/v1/messages \
          -H 'Authorization: Bearer ${WEBEX_TOKEN}' \
          -H 'Content-Type: application/json' \
          -d '{\"roomId\":\"${WEBEX_ROOM}\",\"text\":\"❌ Jenkins build ${JOB_NAME} #${BUILD_NUMBER} FAILED. See: ${BUILD_URL}\"}'
      """
    }
  }
}

