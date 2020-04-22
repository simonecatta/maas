pipeline {
    agent {
        label 'nodejs'
    }
    environment {
        APP_NAME = "simplenodeservice"
        ARTEFACT_ID = "ace/" + "${env.APP_NAME}"
        VERSION = readFile('version').trim()
        TAG = "${env.DOCKER_REGISTRY_URL}/library/${env.ARTEFACT_ID}:${env.VERSION}"
        TAG_DEV = "${env.TAG}-${env.VERSION}-${env.BUILD_NUMBER}"
    }
    stages {
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
                    sh "docker build -t ${env.TAG} ."
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
        /*stage('Mark artifact for staging namespace') {
            steps {
                container('docker'){
                sh "docker tag ${env.TAG_DEV} ${env.TAG_STAGING}"
                sh "docker push ${env.TAG_STAGING}"
                }
            }
        }*/
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