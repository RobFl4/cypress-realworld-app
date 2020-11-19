pipeline {
    agent any

    stages {
        stage('Building') {
            steps {
                echo 'Building..'
            }
        }
        stage('Testing') {
            steps {
                echo 'Testing..'
                sh 'npm run cypress'
            }
        }
        stage('Deploying') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}