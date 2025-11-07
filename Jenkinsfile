pipeline {
  agent { docker { image 'python:3.10-slim'; args '-u root' } }

  environment {
    PYTHONUNBUFFERED = '1'
    WEBEX_BOT_TOKEN  = credentials('webex-bot-token')   // add in Jenkins later
    WEBEX_ROOM_ID    = credentials('webex-room-id')     // add in Jenkins later
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install deps') {
      steps {
        sh '''
          python --version
          pip install -U pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Run unit tests') {
      steps {
        sh '''
          export PYTHONPATH=$PWD
          pytest -q
        '''
      }
    }

    stage('Notify Webex') {
      steps {
        script {
          def msg = "✅ Build: ${env.JOB_NAME} #${env.BUILD_NUMBER} passed. Commit: ${env.GIT_COMMIT?.take(7)}"
          sh """
            curl -s -X POST https://webexapis.com/v1/messages \
              -H "Authorization: Bearer $WEBEX_BOT_TOKEN" \
              -H "Content-Type: application/json" \
              -d '{
                    "roomId": "'$WEBEX_ROOM_ID'",
                    "text": "${msg}"
                  }' >/dev/null
          """
        }
      }
    }
  }

  post {
    failure {
      sh '''
        curl -s -X POST https://webexapis.com/v1/messages \
          -H "Authorization: Bearer '$WEBEX_BOT_TOKEN'" \
          -H "Content-Type: application/json" \
          -d '{"roomId":"'$WEBEX_ROOM_ID'","text":"❌ Build failed: '$JOB_NAME' #'$BUILD_NUMBER'."}' >/dev/null
      '''
    }
  }
}
