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
                nohup 'ls -l' &
            }
        }
        stage('Deploying') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}