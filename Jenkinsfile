node {
    // Use Node 16 Docker image, map port 3000 if needed
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        stage('Install Dependencies') {
            sh '''
                apt-get update && apt-get install -y curl git
                # Install the Heroku CLI
                curl https://cli-assets.heroku.com/install.sh | sh
                export PATH="/usr/local/bin:$PATH"
                heroku --version
                git --version
            '''
        }

        stage('Build') {
            sh 'npm install'
            // If you have a separate build step:
            // sh 'npm run build'
        }

        stage('Test') {
            sh './jenkins/scripts/test.sh'
        }

        stage('Manual Approval') {
            input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
        }

        stage('Deploy to Heroku') {
            sh '''
                export HOME=/root
                export PATH="/usr/local/bin:$PATH"

                HEROKU_API_KEY="HRKU-89e642bc-f4c6-4d5e-bf3a-f456afb1826c"

                # Authenticate with Heroku CLI
                echo "machine api.heroku.com login=heroku password=$HEROKU_API_KEY" > $HOME/.netrc
                echo "machine git.heroku.com login=heroku password=$HEROKU_API_KEY" >> $HOME/.netrc
                chmod 600 $HOME/.netrc

                # Explicitly log in using API key
                heroku auth:token
                heroku whoami

                # Mark Jenkins workspace as safe
                git config --global --add safe.directory "$WORKSPACE"

                cd $WORKSPACE

                # Set up Heroku Git remote
                if ! git remote | grep -q heroku; then
                    heroku git:remote -a react-app-jenkins
                fi

                # Use HTTPS instead of SSH for Git operations
                git remote set-url heroku https://git.heroku.com/react-app-jenkins.git

                # Push the code
                git checkout -B main
                git push -f heroku main
            '''
        }
    }
}
