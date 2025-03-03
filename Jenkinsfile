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

"""
        stage('Build') {
            // Install dependencies (and build if needed)
            sh 'npm install'
            // If you have a separate build step:
            // sh 'npm run build'
        }

        stage('Test') {
            // Run your test script
            sh './jenkins/scripts/test.sh'
        }

        stage('Manual Approval') {
            // Simple manual gate before deployment
            input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
        }
"""

        stage('Deploy to Heroku') {
            // The crucial part
            sh '''
                # Make the Heroku CLI available
                export PATH="/usr/local/bin:$PATH"

                # ---- HARDCODED Heroku API Key (for personal use only) ----
                HEROKU_API_KEY="HRKU-89e642bc-f4c6-4d5e-bf3a-f456afb1826c"

                # Configure .netrc for Heroku authentication
                echo "machine api.heroku.com login=heroku password=$HEROKU_API_KEY" > ~/.netrc
                echo "machine git.heroku.com login=heroku password=$HEROKU_API_KEY" >> ~/.netrc
                chmod 600 ~/.netrc

                # Move into the Jenkins workspace (already mounted in Docker)
                cd $WORKSPACE

                # Make sure we have a .git folder. If not, initialize and do an initial commit.
                if [ ! -d .git ]; then
                  git init
                  git add .
                  git commit -m "Initial commit"
                fi

                # Configure Git (global) for this environment
                git config --global user.email "moch.beta@gmail.com"
                git config --global user.name "mochamdbeta289893"
                git config --global --add safe.directory $WORKSPACE

                # Ensure the 'heroku' remote is set to the correct URL
                if ! git remote | grep -q heroku; then
                  git remote add heroku https://git.heroku.com/react-app-jenkins.git
                else
                  git remote set-url heroku https://git.heroku.com/react-app-jenkins.git
                fi

                # Force-create (or switch to) the 'main' branch
                git checkout -B main

                # Push to Heroku, overwriting if needed
                git push -f heroku main
            '''

            // Wait and check logs
            echo 'Aplikasi berhasil di-deploy ke Heroku. Menunggu 1 menit untuk pengujian...'
            sh 'sleep 60'
            sh 'heroku logs --tail -a react-app-jenkins'
        }
    }
}