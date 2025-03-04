node {
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

        withCredentials([string(credentialsId: 'HEROKU_API_KEY', variable: 'HEROKU_API_KEY')]) {
            stage('Build') {
                sh 'npm install'
            }

            stage('Test') {
                sh './jenkins/scripts/test.sh'
            }

            stage('Manual Approval') {
                input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
            }

            stage('Installing Dependencies') {
                sh '''
                    apt-get update && apt-get install -y curl git
                    curl https://cli-assets.heroku.com/install.sh | sh
                '''
            }            

            stage('Deploy to Heroku') {
                sh '''
                    # Set up API key authentication (Non-Interactive Login)
                    echo "machine api.heroku.com login=heroku password=$HEROKU_API_KEY" > ~/.netrc
                    echo "machine git.heroku.com login=heroku password=$HEROKU_API_KEY" >> ~/.netrc
                    chmod 600 ~/.netrc

                    cat ~/.netrc
                    ls -la ~/.netrc

                    chown -R $(id -u):$(id -g) "$WORKSPACE"
                    git branch -a
                    heroku git:remote -a react-app-jenkins
                    git push heroku main
                '''
            }
        }
    }
}