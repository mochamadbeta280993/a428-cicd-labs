node {
    // Use Node 16 Docker image inside Docker
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        // Fetch Heroku API Key securely from Jenkins credentials
        withCredentials([string(credentialsId: 'HEROKU_API_KEY', variable: 'HEROKU_API_KEY')]) {

            stage('Install Dependencies') {
                sh '''
                    apt-get update && apt-get install -y curl git
                    curl https://cli-assets.heroku.com/install.sh | sh
                    export PATH="/usr/local/bin:$PATH"
                    heroku --version
                    git --version
                '''
            }

            stage('Build') {
                sh 'npm install'
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

                    # Ensure Heroku CLI authentication using API key
                    echo "machine api.heroku.com login=heroku password=$HEROKU_API_KEY" > ~/.netrc
                    echo "machine git.heroku.com login=heroku password=$HEROKU_API_KEY" >> ~/.netrc
                    chmod 600 ~/.netrc

                    # Verify authentication
                    heroku auth:whoami || heroku auth:login --interactive

                    # Set up Git
                    git config --global --add safe.directory "$WORKSPACE"
                    cd $WORKSPACE

                    # Set Heroku Git Remote
                    if ! git remote | grep -q heroku; then
                        heroku git:remote -a react-app-jenkins
                    fi

                    # Explicitly switch Git authentication to use SSH instead of HTTPS
                    git remote set-url heroku git@heroku.com:react-app-jenkins.git

                    # Push code to Heroku
                    git checkout -B main
                    git push -f heroku main
                '''
            }
        }
    }
}