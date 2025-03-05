node {
    // Run the pipeline inside a Docker container using Node.js 16 (Debian Buster Slim)
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        // Use Jenkins credentials to securely retrieve the Heroku API key
        withCredentials([string(credentialsId: 'HEROKU_API_KEY', variable: 'HEROKU_API_KEY')]) {
            
            // **Stage 1: Build**
            stage('Build') {
                sh '''
                    # Install project dependencies from package.json
                    npm install
                '''
            }

            // **Stage 2: Test**
            stage('Test') {
                sh '''
                    # Run the test script (test.sh) inside the jenkins/scripts/ directory
                    ./jenkins/scripts/test.sh
                '''
            }

            // **Stage 3: Manual Approval**
            stage('Manual Approval') {
                // Pause the pipeline and ask for manual approval before deployment
                input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
            }

            // **Stage 4: Installing Dependencies**
            stage('Installing Dependencies') {
                sh '''
                    # Update the package lists
                    apt-get update 

                    # Install curl and Git, required for interacting with Heroku
                    apt-get install -y curl git

                    # Install the Heroku CLI (Command Line Interface)
                    curl https://cli-assets.heroku.com/install.sh | sh
                '''
            }

            // **Stage 5: Deploy to Heroku**
            stage('Deploy to Heroku') {
                sh '''
                    # Change ownership of the workspace to the current Jenkins user
                    chown -R $(id -u):$(id -g) "$WORKSPACE"

                    # Set the Heroku Git remote repository
                    heroku git:remote -a react-app-jenkins

                    # Set OpenSSL legacy mode before pushing
                    heroku config:set NODE_OPTIONS=--openssl-legacy-provider -a react-app-jenkins

                    # Push to Heroku
                    git push https://heroku:$HEROKU_API_KEY@git.heroku.com/react-app-jenkins.git main
					
					# Start the application in Heroku
					heroku ps:scale web=1 -a react-app-jenkins
					
					# Pause the pipeline execution for 1 minute
					sleep 60

					# After waiting for 1 minute, stop the application in Heroku
					heroku ps:scale web=0 -a react-app-jenkins
                '''
            }
        }
    }
}