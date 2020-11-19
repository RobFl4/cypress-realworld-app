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
                sh 'yarn cypress:run'
            }
        }
        stage('Deploying') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}