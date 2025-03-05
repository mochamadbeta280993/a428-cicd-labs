node {
    // Run the pipeline inside a Docker container using Node.js 16 (Debian Buster Slim)
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        // Use Jenkins credentials to securely retrieve the Heroku API key
        withCredentials([string(credentialsId: 'HEROKU_API_KEY', variable: 'HEROKU_API_KEY')]) {
            
            // **Stage 1: Build**
            stage('Build') {
                // Install project dependencies
                sh 'npm install'
            }

            // **Stage 2: Test**
            stage('Test') {
                // Run the test script
                sh './jenkins/scripts/test.sh'
            }

            // **Stage 3: Manual Approval**
            stage('Manual Approval') {
                // Pause the pipeline and ask for manual approval before deployment
                input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
            }

            // **Stage 4: Installing Dependencies**
            stage('Installing Dependencies') {
                // Update the package lists
                sh 'apt-get update'

                // Install curl and Git
                sh 'apt-get install -y curl git'
                
                // Install the Heroku CLI
                sh 'curl https://cli-assets.heroku.com/install.sh | sh'
            }

            // **Stage 5: Deploy to Heroku**
            stage('Deploy to Heroku') {
                // Change ownership of workspace
                sh 'chown -R $(id -u):$(id -g) "$WORKSPACE"'

                // Set the Heroku Git remote
                sh 'heroku git:remote -a react-app-jenkins'

                // Set OpenSSL legacy mode before pushing
                sh 'heroku config:set NODE_OPTIONS=--openssl-legacy-provider -a react-app-jenkins'

                // Push to Heroku
                sh 'git push https://heroku:$HEROKU_API_KEY@git.heroku.com/react-app-jenkins.git main'

                // Start the application in Heroku
                sh 'heroku ps:scale web=1 -a react-app-jenkins'
            }

            // **Stage 6: Wait and Kill**
            stage('Wait and Kill') {
                // Pause the pipeline execution for 1 minute
                sh 'sleep 60'

                // Stop the application in Heroku
                sh 'heroku ps:scale web=0 -a react-app-jenkins'
            }
        }
    }
}