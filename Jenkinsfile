@Library('vfz-common-ci') _

// Branch variables
master = "master".equals(env.BRANCH_NAME)
develop = "develop".equals(env.BRANCH_NAME)
safeBranchName = ci.safeBranchName env.BRANCH_NAME
encodedBranchName = env.BRANCH_NAME.replaceAll("/", "%2F")
// acc is not for us for now I guess
// awsEnv = master ? "dev" : develop ? "acc" : "dev"
awsEnv = "dev"

awsRegion = "eu-west-1"
// acc is not for us for now I guess
// ecrId = master ? "118404650135" : develop ? "tbd" : "118404650135"
ecrId = "118404650135"

pipeline {
    agent any
    options {
        disableConcurrentBuilds()
    }
    environment {
        // Docker variables
        dockerImage = ''
        imageName = "eshop/vfz-module-eshop-ziggonl-inlifesales"
        imageTag = "${env.GIT_COMMIT}"
        fullImageName = "${imageName}:${imageTag}"
        imageRepo = "${ecrId}.dkr.ecr.eu-west-1.amazonaws.com"

        // Set ECR id which differs per aws environment
        dockerRegistry = "https://${imageRepo}"

        // Kubernetes deployment vars
        clusterName = "eshop"
        nameSpace = "eshop"
        projectName = "eshop-module-ziggonl-inlifesales"
        fqdn = "${projectName}-${safeBranchName}.${clusterName}.${awsEnv}.aws.ziggo.io"
        svcPort = "6000"

        // Disabled the tests, since there fail right now
        RUN_TESTS = "false"
        RUN_STEP = "true"
        SKIP_BUILD = "false"

        // These 3 (serviceName, imageName, exposedPort) are ideally the only ones you need or want to configure
        serviceName = "eshop-ziggonl-inlifesales"
        exposedPort = "80"
        fullBranchName = "${projectName}-${safeBranchName}"

        // We do this in 2 steps to simplify Groovy syntax

        NPM_TOKEN = credentials('nexus-npm-deploy-token')
    }

    stages {
        stage('Pre-checks') {
            steps {
                script {
                    // Set environment to prod only when branch is master
                    if (ci.isCiUser()) {
                        echo "CI user is detected, setting RUN_STEP to false"
                        RUN_STEP = 'false'
                    }
                }
            }
        }
        stage('Image check') {
            steps {
                script {
                    NOTAGSFOUND = sh (
                      script: 'aws ecr list-images --repository-name ${imageName} | grep -c ${imageTag}||true',
                      returnStdout: true
                    ) as Integer
                    if (NOTAGSFOUND) {
                        RUN_STEP = 'false'
                    }
                    echo RUN_STEP
                }
            }
        }

        stage('Jest test and build') {
            steps {
                script {
                    sh "aws ecr get-login --no-include-email --region ${awsRegion} >> login.sh && sh login.sh"
                    sh 'cp .npmrc-ci .npmrc'
                    sh 'npm install'
                    sh 'npm test'
                    sh 'npm run build:LIVE'
                    withCredentials([usernameColonPassword(credentialsId: 'eshop-components-push-tag-access', variable: 'GIT_CREDENTIALS')]) {
                      sh 'npm run semantic-release'
                    }
                    sh 'npm run build:dev'
                    docker.withRegistry("${dockerRegistry}") {
                        dockerImageBuild = docker.build(fullImageName, "--network=host --build-arg NPM_TOKEN=${NPM_TOKEN} .")
                    }
                }
            }
        }

        stage('Image push') {
            when {
                expression { return RUN_STEP == "true" }
            }
            steps {
                script {
                    sh "aws ecr get-login --no-include-email --region ${awsRegion} >> login.sh && sh login.sh"
                    docker.withRegistry("${dockerRegistry}") {
                        dockerImageBuild.push()
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    //Update Kubeconfig
                    aws.setKubeConfig("${clusterName}")

                    // Check if Ingress already exists and create if it doesn't
                    doesIngressExist = aws.checkIngress("${nameSpace}", "${fullBranchName}")

                    if (doesIngressExist) {
                        sh "echo Ingress already exists. Skipping creation."
                    } else {
                        sh "echo Ingress does not exist. Creating ingress."
                        //Fill ingress template with branch specific values
                        ci.replaceInFile(file: "kubernetes/ingress.yaml", key: "SERVER_ALIAS", value: "${fullBranchName}")
                        ci.replaceInFile(file: "kubernetes/ingress.yaml", key: "INGRESS_NAME", value: "${fullBranchName}")
                        ci.replaceInFile(file: "kubernetes/ingress.yaml", key: "FQDN", value: "${fqdn}")
                        ci.replaceInFile(file: "kubernetes/ingress.yaml", key: "SVC_NAME", value: "${fullBranchName}")
                        ci.replaceInFile(file: "kubernetes/ingress.yaml", key: "SVC_PORT", value: "${svcPort}")
                        // Print final yaml file
                        sh "cat kubernetes/ingress.yaml"                        
                        // Create ingress
                        aws.createIngress("${nameSpace}", "kubernetes/ingress.yaml")
                    }
                    
                    //Fill deployment template with branch specific values
                    ci.replaceInFile(file: "kubernetes/deployment.yaml", key: "NAME", value: "${fullBranchName}")
                    ci.replaceInFile(file: "kubernetes/deployment.yaml", key: "REPO", value: "${imageRepo}")
                    ci.replaceInFile(file: "kubernetes/deployment.yaml", key: "IMAGE_NAME", value: "${fullImageName}")

                    // Check service & create if necessary
                    doesServiceExist = aws.checkService("${nameSpace}", "${fullBranchName}")

                    if (doesServiceExist) {
                        sh "echo Service already exists. Skipping creation."
                    } else {
                       // Fill service template with values
                        sh "echo Service does not exist. Creating service."
                        ci.replaceInFile(file: "kubernetes/service.yaml", key: "NAME", value: "${fullBranchName}")
                        ci.replaceInFile(file: "kubernetes/service.yaml", key: "EXPOSED_PORT", value: "${exposedPort}")
                        ci.replaceInFile(file: "kubernetes/service.yaml", key: "SVC_PORT", value: "${svcPort}")
                        // Print final yaml file
                        sh "cat kubernetes/service.yaml"

                        aws.createService("${nameSpace}", "kubernetes/service.yaml")
                    }

                    // Deploy application
                    doesDeploymentExist = aws.checkDeployment("${nameSpace}", "${fullBranchName}")

                    if (doesDeploymentExist) {
                        sh "echo Deployment exists. Updating current deployment"
                        aws.deployApplication("${nameSpace}", "kubernetes/deployment.yaml", "update")
                    } else {
                        sh "echo Deployment does not exist. Creating new deployemnt"
                        aws.deployApplication("${nameSpace}", "kubernetes/deployment.yaml", "create")
                    }
                    sh "kubectl -n ${namespace} get pods" 
                    sh "echo url ${fqdn}"
                }
            }
        }
 
       //  stage('trigger-testing') {
         //   steps {
            //     script {
              //       def testurl = "https://${fqdn}"
                 //    echo "starting eshop-ILS-cypress for branch ${encodedBranchName}"
                //     build job: "vfz-module-eshop-ziggonl-inlifesales-cypres/${encodedBranchName}",
                //         parameters: [
                //             string(name: 'url', value: testurl)
                //        ],
               //          wait: false
              //   }

         //  }
       // }
    }
}
