node {
    // Define the Docker image
    def nodeImage = docker.image('node:16-buster-slim')

    // Run the Docker container and execute commands inside it
    nodeImage.inside('-p 3000:3000') {
        stage('Build') {
            // Install dependencies
            sh 'npm install'
        }
        stage('Test') {
            // Run tests
            sh './jenkins/scripts/test.sh'
        }
    }
}