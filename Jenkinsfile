pipeline {
	agent {
		docker {
			image 'node:16-buster-slim'
			args '-p 3000:3000'
		}
	}
	stages {
		stage('Build') {
			steps {
				sh 'npm install'
			}
		}
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Deploy') {
            steps {
                sh './jenkins/scripts/deliver.sh'
                input message: 'Sudah selesai menggunakan React App? (Klik "Proceed" untuk mengakhiri)'
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}

"""
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
"""