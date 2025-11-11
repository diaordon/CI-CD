pipeline {
  agent any

  options { timestamps() }

  environment {
    // Jenkins credential: Secret text with your bot token (ID must match)
    WEBEX_TOKEN = credentials('webex-token')
    // <-- put your Webex Space (room) ID here
    WEBEX_ROOM  = 'Y2lzY29zcGFyazovL3VybjpURUFNOnVzLXdlc3QtMl9yL1JPT00vNDA4ZjVkNjAtYmU5NC0xMWYwLTg3MGMtYmIzN2M0ZWE4MTVk'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install & Test') {
      steps {
        sh '''#!/usr/bin/env bash
set -euxo pipefail

python3 --version

# Create and activate a virtual environment (avoids PEP 668 errors)
python3 -m venv .venv
. .venv/bin/activate

python -m pip install --upgrade pip
if [ -f requirements.txt ]; then
  pip install -r requirements.txt
fi

# Helpful visibility in Jenkins logs
echo "PWD=$(pwd)"
ls -la
find . -maxdepth 2 -type f -print

# Ensure your workspace root (where calculator.py lives) is importable
export PYTHONPATH="$PWD"

# Run tests and always write JUnit XML for Jenkins
mkdir -p test-results
python -m pytest -q --junitxml=test-results/pytest.xml
'''
      }
    }
  }

  post {
    always {
      // publish test report (even on failure) and keep the XML
      junit 'test-results/pytest.xml'
      archiveArtifacts artifacts: 'test-results/pytest.xml', onlyIfSuccessful: false
    }

    success {
      sh '''#!/usr/bin/env bash
curl -sS https://webexapis.com/v1/messages \
  -H "Authorization: Bearer ${WEBEX_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"roomId":"'"${WEBEX_ROOM}"'","text":"✅ Jenkins build '"${JOB_NAME}"' #'"${BUILD_NUMBER}"' SUCCESS on '"${GIT_BRANCH}"'"}' >/dev/null
'''
    }

    failure {
      sh '''#!/usr/bin/env bash
curl -sS https://webexapis.com/v1/messages \
  -H "Authorization: Bearer ${WEBEX_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"roomId":"'"${WEBEX_ROOM}"'","text":"❌ Jenkins build '"${JOB_NAME}"' #'"${BUILD_NUMBER}"' FAILED. See: '"${BUILD_URL}"'"}' >/dev/null
'''
    }
  }
}

