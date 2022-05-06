pipeline {
    agent any
    tools {nodejs "Node"}
    environment {  
        DEPLOY = 'false'
    }
     parameters { choice(name: 'DEV_ENV', choices: ['pde-dev'], description: '')
                  choice(name: 'API_VER', choices: ['v1', 'v2', 'v3'], description: '')
     }
    stages {

    stage('Setup Environment for APICTL') {
            steps {
                script {
                if( env.DEV_ENV == "pde-dev") {
                    HOST = "dev-am.puertos.es"
                    }
                } 
                sh 'apictl version'
                sh """#!/bin/bash
                ENVCOUNT=\$(apictl list envs --format {{.}} | grep $DEV_ENV | wc -l)
                if [ "\$ENVCOUNT" == "0" ]; then
                    apictl add-env -e $DEV_ENV --apim https://$HOST
                fi
                """
            }
        }
        stage('Deploy to API MANAGER') {
            environment{
                RETRY = '80'
            }
            steps {
                echo "Logging into ${DEV_ENV}"
                echo "API Version ${API_VER}"
                withCredentials([usernamePassword(credentialsId: "devops", usernameVariable: 'DEV_USERNAME', passwordVariable: 'DEV_PASSWORD')]) {
                    sh "apictl login ${DEV_ENV} -u $DEV_USERNAME -p $DEV_PASSWORD -k"
                }
               
                echo 'Deploying to $DEV_ENV'
             
                sh 'apictl import-api -f ./$API_VER -e $DEV_ENV -k --preserve-provider --update --verbose'
                /*
                sh 'apictl vcs deploy -e $DEV_ENV -k'
                
                */
            }
        }
        /*
        stage('Run Tests') {
            steps {
                echo 'Running tests in $DEV_ENV'
                sh 'newman run ./postman/PoC.postman_collection.json -e ./postman/$DEV_ENV.json --reporters cli,junit,htmlextra --reporter-junit-export "postman.xml" -n 1 --reporter-htmlextra-export "newman/postman.html" --insecure --bail' 
              post{
            success {
                junit '*.xml'
                publishHTML (target: [
                  allowMissing: false,
                  alwaysLinkToLastBuild: false,
                  keepAll: true,
                  reportDir: 'newman',
                  reportFiles: 'postman.html',
                  reportName: "postman-test"
                ])
             }
            failure {
                
                junit '*.xml'
                publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'newman',
                reportFiles: 'report.html',
                reportName: "postman-test"
               ])
        }
            }
        
            }
        }
        */
    }
    post {
        cleanup {
            deleteDir()
            dir("${workspace}@tmp") {
                deleteDir()
            }
            dir("${workspace}@script") {
                deleteDir()
            }
        }
    }
}
