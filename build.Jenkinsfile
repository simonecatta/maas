pipeline {
    parameters {
        choice(name: 'BUILD', choices: ['1','2','3','4'], description: 'Select the build you want to deploy (affects application behavior, github.com/grabnerandi/simplenodeservice for more details)')
    }
    environment {
        APP_NAME = "simplenodeservice"
        ARTEFACT_ID = "ace/" + "${env.APP_NAME}"
        TAG = "${env.DOCKER_REGISTRY_URL}/library/${env.ARTEFACT_ID}:${env.BUILD}.0.0-${env.BUILD_NUMBER}"
    }
    agent {
        label 'nodejs'
    }
    stages {
        node ('nodejs') {
            stage('Node build') {
                steps {
                    checkout scm
                    container('nodejs') {
                    sh 'npm install'
                    }
                }
            } 
            stage('Docker build') {
                steps {
                    container('docker') {
                        sh "docker build --build-arg BUILD_NUMBER=${env.BUILD} -t ${env.TAG} ."
                    }
                }
            }
            stage('Docker push to registry') {
                steps {
                    container('docker') {
                        sh "docker push ${env.TAG}"
                    }
                }
            }
        }
        node ('master') {
            stage('Deploy to staging') {
                steps {
                    build job: "2. Deploy",
                    parameters: [
                        string(name: 'APP_NAME', value: "${env.APP_NAME}"),
                        string(name: 'TAG_STAGING', value: "${env.TAG}"),
                        string(name: 'VERSION', value: "${env.VERSION}")
                    ]
                }
            }  
        }
    }
}