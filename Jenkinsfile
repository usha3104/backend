pipeline {
  agent any

  environment {
    APP_ID = "fvyTy-oV8pJowrPFyl3Ec"
  }

  parameters {
    string(name: 'DEPLOY_URL_CRED_ID', defaultValue: 'DEPLOY_URL', description: 'Credentials ID for the deployment URL (Secret Text)')
    string(name: 'DEPLOY_KEY_CRED_ID', defaultValue: 'DEPLOY_KEY', description: 'Credentials ID for the deployment API key (Secret Text)')
    string(name: 'VITE_CANDIDATES_ENDPOINT', defaultValue: 'VITE_CANDIDATES_ENDPOINT', description: 'Endpoint for candidates API used by the frontend (exported into .env)')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup .env') {
      steps {
        sh "echo 'VITE_CANDIDATES_ENDPOINT=${params.VITE_CANDIDATES_ENDPOINT}' > .env"
        echo ".env created with VITE_CANDIDATES_ENDPOINT"
      }
    }

    stage('Install dependencies') {
      steps {
        sh '''
        node -v
        npm --version
        if [ -f package.json ]; then
          echo "package.json found. Installing dependencies..."
          npm ci || npm install
        else
          echo "⚠️ No package.json found. Skipping npm install."
        fi
        '''
      }
    }

    stage('Run tests') {
      steps {
        sh '''
        if [ -f package.json ] && npm run | grep -q "test"; then
          echo "Running tests..."
          npm test -- --run
        else
          echo "⚠️ No tests found. Skipping test stage."
        fi
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Tests passed (or skipped). Triggering deployment..."

      withCredentials([
        string(credentialsId: params.DEPLOY_URL_CRED_ID, variable: 'DEPLOY_URL'),
        string(credentialsId: params.DEPLOY_KEY_CRED_ID, variable: 'DEPLOY_KEY')
      ]) {
        sh '''
        json_payload=$(printf '{"applicationId":"%s"}' "$APP_ID")

        curl -fS -X POST "$DEPLOY_URL" \
          -H 'accept: application/json' \
          -H 'Content-Type: application/json' \
          -H "x-api-key: $DEPLOY_KEY" \
          --data-binary "$json_payload" \
          -w "\nHTTP %{http_code}\n"
        '''
      }

      mail to: 'ushammusha95@gmail.com',
           subject: "✅ Jenkins Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) succeeded.

• Branch: ${env.BRANCH_NAME}
• Commit: ${env.GIT_COMMIT}
• Build URL: ${env.BUILD_URL}

Deployment API triggered.

Regards,
Jenkins
"""
    }

    failure {
      echo "❌ Pipeline failed. Sending failure email..."

      mail to: 'ushammusha95@gmail.com',
           subject: "❌ Jenkins Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) failed.

• Branch: ${env.BRANCH_NAME}
• Commit: ${env.GIT_COMMIT}
• Build URL: ${env.BUILD_URL}

Check console logs for details.

Regards,
Jenkins
"""
    }
  }
}
