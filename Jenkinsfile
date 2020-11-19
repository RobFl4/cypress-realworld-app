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
                sh 'DISPLAY= xvfb-run -a ./node_modules/.bin/cypress run'
            }
        }
        stage('Deploying') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}