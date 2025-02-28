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
                // Run the deliver.sh script to start the application
                sh './jenkins/scripts/deliver.sh'

                // Display a message to let you know Jenkins is "waiting" for 1 minute
                echo 'Aplikasi berhasil di-deploy. Menunggu 1 menit untuk pengujian...'

                // Pause the pipeline execution for 60 seconds
                sh 'sleep 60'

                // After 1 minute, stop the application
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