node { 
    docker.image('node:16-buster-slim').inside('--user root -p 3000:3000') {

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
            export PATH="/usr/local/bin:$PATH"
            HEROKU_API_KEY="HRKU-89e642bc-f4c6-4d5e-bf3a-f456afb1826c"
            echo "machine api.heroku.com login=heroku password=$HEROKU_API_KEY" > ~/.netrc
            echo "machine git.heroku.com login=heroku password=$HEROKU_API_KEY" >> ~/.netrc
            chmod 600 ~/.netrc

            heroku git:remote -a react-app-jenkins
            git config --global --add safe.directory /var/jenkins_home/workspace/react-app
            git config --global user.email "moch.beta@gmail.com"
            git config --global user.name "mochamadbeta280993"

            git add .
            git commit -m "Deploy from Jenkins"
            git push heroku main
            '''

            echo 'Aplikasi berhasil di-deploy ke Heroku. Menunggu 1 menit untuk pengujian...'

            sh 'sleep 60'

            sh 'heroku logs --tail -a react-app-jenkins'
        }
    }
}