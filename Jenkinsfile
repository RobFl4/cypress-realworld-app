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
                yarn --version
                yarn cypress:run
            }
        }
        stage('Deploying') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}