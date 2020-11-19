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
                cypress run
                // sh 'DISPLAY= xvfb-run -a ./node_modules/.bin/cypress run' +
                //    '--browser chrome' +
                //    '--config baseUrl=https://http://localhost:3000/' +
                //    '--spec=./cypress/tests/ui/auth.spec.ts'
            }
            // 
                    //             steps {
                    //     script {
                    //         dir('test'){
                    //             sh "mv ../.npmrc-ci .npmrc"
                    //             sh "npm config set strict-ssl false"
                    //             sh "npm install"
                    //             sh "npm run copy"
                    //             sh "DISPLAY= xvfb-run -a ./node_modules/.bin/cypress run " +
                    //                // "--record --key 35c676b0-f876-4692-9dc8-52996b53d06f --parallel " +
                    //                "--record --key 35c676b0-f876-4692-9dc8-52996b53d06f " +
                    //                "--browser chrome " +
                    //                "--config baseUrl=https://eshop-module-ziggonl-inlifesales-${safeBranchName}.eshop.dev.aws.ziggo.io " +
                    //                "--spec=./cypress/tests/integration/*.js"
                    //         }
                    //     }
                    // }
            // 
        }
        stage('Deploying') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}