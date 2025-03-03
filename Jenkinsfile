node {
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        stage('Install Dependencies') {
            sh '''
                apt-get update && apt-get install -y curl git
                # Install Heroku CLI
                curl https://cli-assets.heroku.com/install.sh | sh
                export PATH="/usr/local/bin:$PATH"
                heroku --version
                git --version
            '''
        }

        stage('Build') {
            sh 'npm install'
            // If you need a production build, you might also do:
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
                # Make Heroku CLI available
                export PATH="/usr/local/bin:$PATH"
                
                # ---- HARDCODED Heroku API Key (Not recommended for production) ----
                HEROKU_API_KEY="HRKU-89e642bc-f4c6-4d5e-bf3a-f456afb1826c"
                
                # Set up .netrc for Heroku
                echo "machine api.heroku.com login=heroku password=$HEROKU_API_KEY" > ~/.netrc
                echo "machine git.heroku.com login=heroku password=$HEROKU_API_KEY" >> ~/.netrc
                chmod 600 ~/.netrc

                # Go to the Jenkins workspace
                cd $WORKSPACE

                # Initialize Git if not already
                if [ ! -d .git ]; then
                    git init
                    git remote add heroku https://git.heroku.com/react-app-jenkins.git
                else
                    git remote set-url heroku https://git.heroku.com/react-app-jenkins.git
                fi

                # Git config
                git config --global user.email "moch.beta@gmail.com"
                git config --global user.name "mochamdbeta289893"
                git config --global --add safe.directory $WORKSPACE

                # Stage and commit changes (ignore "nothing to commit" errors)
                git add .
                git commit -m "Deploy from Jenkins" || echo "No changes to commit"

                # Force-checkout (create/replace) main branch
                git checkout -B main

                # Push code to Heroku
                git push -f heroku main
            '''

            echo 'Aplikasi berhasil di-deploy ke Heroku. Menunggu 1 menit untuk pengujian...'
            sh 'sleep 60'
            sh 'heroku logs --tail -a react-app-jenkins'
        }
    }
}