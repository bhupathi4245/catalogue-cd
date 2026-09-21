pipeline {
    agent {
        label 'AGENT-1'
    }
    environment {
        // Environment variables
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '123456789012'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }
    options {
        // Options
        buildDiscarder(logRotator(numToKeepStr: '5'))
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    // Build section (Out of 3 sections: Pre-Build, Build, Post-Build)
    stages {
        stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: ${REGION}) {
                        sh """
                            aws eks update-kubeconfig --name "${PROJECT}-${params.deploy_to}" --region ${REGION}
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "/sIMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            help upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n ${PROJECT} .
                        """
                    }
                }
            }
        }

        stage('Check Status') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: ${REGION}) {
                        def deploymentStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --timeout=30s || echo FAILED").trim()
                        if (deploymentStatus.contains("successfully rolled out")) {
                            echo "Deployment successful."              
                        } else {
                            sh """
                                helm rollback $COMPONENT 1 -n ${PROJECT}
                                sleep 20
                            """
                            def rollbackStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --timeout=30s || echo FAILED").trim()
                            if (rollbacktStatus.contains("successfully rolled out")) {
                                error "Deployment is Failure, Rollback success."
                            } else {
                            error "Deployment failed. Rollback failed. Application is not in a stable state."
                            }
                        }
                    }
                }
            }
        }
    }
}

    // post section
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir() // Clean up our workspace
        }
        success { 
            echo 'Hello again! Success'
        }
        failure { 
            echo 'Hello again! Failure'
        }
    }
}