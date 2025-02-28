node {
    // Pull and run the specified Docker container, exposing port 3000
    docker.image('node:16-buster-slim').inside('-p 3000:3000') {

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

        // Stage: Deploy
        stage('Deploy') {
            // Run the deliver.sh script to start the application
            sh './jenkins/scripts/deliver.sh'

            // Display a message indicating a 1-minute wait for testing
            echo 'Aplikasi berhasil di-deploy. Menunggu 1 menit untuk pengujian...'

            // Pause the pipeline execution for 60 seconds
            sh 'sleep 60'

            // After waiting for 1 minute, stop the application
            sh './jenkins/scripts/kill.sh'
        }
    }
}