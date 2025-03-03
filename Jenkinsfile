node { 
    // Run inside a Node.js container with root access
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        // Define environment variables
        environment {
            HEROKU_API_KEY = 'HRKU-89e642bc-f4c6-4d5e-bf3a-f456afb1826c' // Heroku API Key
            HEROKU_APP_NAME = 'react-app-jenkins' // Heroku App Name
        }

        // Install necessary dependencies and Heroku CLI
        stage('Install Heroku CLI') {
            sh '''
            apt-get update && apt-get install -y curl
            curl https://cli-assets.heroku.com/install.sh | sh
            export PATH="/usr/local/bin:$PATH"
            heroku --version
            '''
        }

        // Install project dependencies
        stage('Build') {
            sh 'npm install'
        }

        // Run tests
        stage('Test') {
            sh './jenkins/scripts/test.sh'
        }

        // Pause for manual approval before deploying
        stage('Manual Approval') {
            input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
        }

        // Deploy the React app to Heroku
        stage('Deploy to Heroku') {
            sh '''
            export PATH="/usr/local/bin:$PATH" // Ensure Heroku CLI is in PATH
            echo $HEROKU_API_KEY | heroku auth:token // Authenticate with Heroku
            heroku git:remote -a $HEROKU_APP_NAME // Set Heroku Git remote
            git add . // Stage changes
            git commit -m "Deploy from Jenkins" // Commit changes
            git push heroku main // Push to Heroku
            '''

            echo 'Aplikasi berhasil di-deploy ke Heroku. Menunggu 1 menit untuk pengujian...'

            sh 'sleep 60' // Wait for 1 minute before checking logs

            sh 'heroku logs --tail -a $HEROKU_APP_NAME' // Display Heroku logs
        }
    }
}