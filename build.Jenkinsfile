pipeline {
    agent {
        label 'nodejs'
    }
    environment {
        APP_NAME = "simplenodeservice"
        ARTEFACT_ID = "ace/" + "${env.APP_NAME}"
        VERSION = "1.0"
        TAG = "${env.DOCKER_REGISTRY_URL}:5000/library/${env.ARTEFACT_ID}"
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
    }
    stage('Docker build') {
        steps {
            container('docker') {
                sh "docker build -t ${env.TAG_DEV} ."
            }
        }
    }
    stage('Docker push to registry') {
        steps {
            container('docker') {
            sh "docker push ${env.TAG_DEV}"
            }
        }
    }
}