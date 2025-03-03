node {
    // Pull and run the specified Docker container, exposing port 3000
    docker.image('node:16-buster-slim').inside('-p 3000:3000') {

        environment {
            HEROKU_API_KEY = 'HRKU-89e642bc-f4c6-4d5e-bf3a-f456afb1826c'
            HEROKU_APP_NAME = 'react-app-jenkins'
        }

        // Stage: Build
        stage('Build') {
            // Install project dependencies using npm
            sh 'npm install'
        }

        // Stage: Test
        stage('Test') {
            // Execute the test script stored in ./jenkins/scripts/test.sh
            sh './jenkins/scripts/test.sh'
        }

        // Stage: Manual Approval
        stage('Manual Approval') {
            // Prompt user to continue to Deploy stage
            input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
        }

        // Stage: Deploy to Heroku
        stage('Deploy to Heroku') {
            // Authenticate with Heroku
            sh 'export PATH=$PATH:/usr/local/bin'
            sh 'heroku git:remote -a $HEROKU_APP_NAME'
            sh 'git add .'
            sh 'git commit -m "Deploy from Jenkins"'
            sh 'git push heroku main'

            // Display a message indicating a 1-minute wait for testing
            echo 'Aplikasi berhasil di-deploy ke Heroku. Menunggu 1 menit untuk pengujian...'

            // Pause the pipeline execution for 60 seconds
            sh 'sleep 60'

            // Check logs for issues
            sh 'heroku logs --tail -a $HEROKU_APP_NAME'
        }
    }
}