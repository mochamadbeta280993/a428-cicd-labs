node {
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        withCredentials([string(credentialsId: 'HEROKU_API_KEY', variable: 'HEROKU_API_KEY')]) {

            stage('Install Dependencies') {
                sh '''
                    apt-get update && apt-get install -y curl git openssh-client
                    curl https://cli-assets.heroku.com/install.sh | sh
                    export PATH="/usr/local/bin:$PATH"
                    heroku --version
                    git --version
                '''
            }

            stage('Generate SSH Key') {
                sh '''
                    # Set up SSH for Heroku Deployment
                    mkdir -p ~/.ssh
                    ssh-keygen -t rsa -b 4096 -C "jenkins@ci" -f ~/.ssh/id_rsa -N ""
                    eval $(ssh-agent -s)
                    ssh-add ~/.ssh/id_rsa

                    # Display Public Key (for debugging)
                    cat ~/.ssh/id_rsa.pub
                '''
            }

            stage('Add SSH Key to Heroku') {
                sh '''
                    export HOME=/root
                    export PATH="/usr/local/bin:$PATH"

                    # Authenticate with Heroku using API Key
                    echo "machine api.heroku.com login=heroku password=$HEROKU_API_KEY" > ~/.netrc
                    echo "machine git.heroku.com login=heroku password=$HEROKU_API_KEY" >> ~/.netrc
                    chmod 600 ~/.netrc

                    # Add the generated SSH key to Heroku
                    heroku keys:add ~/.ssh/id_rsa.pub --yes

                    # Verify SSH connection to Heroku
                    ssh -o StrictHostKeyChecking=no -T git@heroku.com || echo "SSH connection to Heroku failed"
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

                    # Configure Git to use SSH for Heroku
                    git config --global user.email "jenkins@ci"
                    git config --global user.name "Jenkins CI"
                    git config --global --add safe.directory "$WORKSPACE"
                    
                    cd $WORKSPACE

                    # Set Heroku Git Remote
                    if ! git remote | grep -q heroku; then
                        heroku git:remote -a react-app-jenkins
                    fi

                    # Use SSH instead of HTTPS
                    git remote set-url heroku git@heroku.com:react-app-jenkins.git

                    # Push code to Heroku
                    git checkout -B main
                    GIT_SSH_COMMAND="ssh -o StrictHostKeyChecking=no" git push -f heroku main
                '''
            }
        }
    }
}
