node {
    // Use Node 16 Docker image, map port 3000 if needed
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        stage('Install Dependencies') {
            sh '''
                apt-get update && apt-get install -y curl git openssh-client
                # Install the Heroku CLI
                curl https://cli-assets.heroku.com/install.sh | sh
                export PATH="/usr/local/bin:$PATH"
                heroku --version
                git --version
            '''
        }

        // Commenting out the Build stage
        /*
        stage('Build') {
            // Install dependencies (and build if needed)
            sh 'npm install'
            // If you have a separate build step:
            // sh 'npm run build'
        }
        */

        // Commenting out the Test stage
        /*
        stage('Test') {
            // Run your test script
            sh './jenkins/scripts/test.sh'
        }
        */

        // Commenting out the Manual Approval stage
        /*
        stage('Manual Approval') {
            // Simple manual gate before deployment
            input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
        }
        */

        stage('Deploy to Heroku') {
            // The crucial part
            sh '''
                # Make the Heroku CLI available
                export PATH="/usr/local/bin:$PATH"

                # ---- HARDCODED Heroku API Key (for personal use only) ----
                HEROKU_API_KEY="HRKU-89e642bc-f4c6-4d5e-bf3a-f456afb1826c"

                # Authenticate Heroku CLI non-interactively using the API key
                heroku auth:token --no-warn > /dev/null 2>&1
                echo $HEROKU_API_KEY | heroku auth:login --no-warn

                # Move into the Jenkins workspace (already mounted in Docker)
                cd $WORKSPACE

                # Ensure Git is initialized and configured
                if [ ! -d .git ]; then
                  git init
                  git add .
                  git commit -m "Initial commit"
                fi

                # Configure Git (global) for this environment
                git config --global user.email "moch.beta@gmail.com"
                git config --global user.name "mochamdbeta289893"
                git config --global --add safe.directory $WORKSPACE

                # Add Heroku remote using SSH
                HEROKU_APP_NAME="react-app-jenkins"
                HEROKU_GIT_URL="git@heroku.com:$HEROKU_APP_NAME.git"

                if ! git remote | grep -q heroku; then
                  git remote add heroku $HEROKU_GIT_URL
                else
                  git remote set-url heroku $HEROKU_GIT_URL
                fi

                # Ensure SSH keys are set up
                ssh-keyscan -t rsa git.heroku.com >> ~/.ssh/known_hosts

                # Force-create (or switch to) the 'main' branch
                git checkout -B main

                # Push to Heroku
                git push heroku main
            '''

            // Wait and check logs
            echo 'Aplikasi berhasil di-deploy ke Heroku. Menunggu 1 menit untuk pengujian...'
            sh 'sleep 60'
            sh 'heroku logs --tail -a react-app-jenkins'
        }
    }
}