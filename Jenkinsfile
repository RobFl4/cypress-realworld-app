// The pipeline block consists of all the instructions to build, test, and deliver software. 
//     It is the key component of a Jenkins Pipeline.
// An agent is assigned to execute the pipeline on a node and allocate a workspace for the pipeline.
// A stage is a block that has steps to build, test, and deploy the application. 
//     Stages are used to visualize the Jenkins Pipeline processes.
// A step is a single task to be performed, for example, create a directory, run a docker image, delete a file, etc.

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                // echo 'Testing..'
                npm i
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}