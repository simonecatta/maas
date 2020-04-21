pipeline {
    parameters {
        string(name: 'APP_NAME', defaultValue: 'simplenodeservice', description: 'The name of the service to deploy.', trim: true)
        string(name: 'TAG_STAGING', defaultValue: 'grabnerandi/simplenodeservice:1.0.0', description: 'The image of the service to deploy.', trim: true)
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'The version of the service to deploy.', trim: true)
        string(name: 'DT_CUSTOM_PROP', defaultValue: '', description: 'Custom properties to be supplied to Dynatrace.', trim: true)
        choice(name: 'QUALITYGATE_PROVIDER', choices: ['Performance Signature Plugin','Keptn Quality Gates', 'None'], description: 'Select your evaluation provider. \'None\' will not evaluate')
    }
    agent {
        label 'kubegit'
    }
    environment {
        APP_NAME = "simplenodeservice"
        ARTEFACT_ID = "ace/" + "${env.APP_NAME}"
        VERSION = "1.0"
        TAG = "${env.DOCKER_REGISTRY_URL}:5000/library/${env.ARTEFACT_ID}"
        TAG_DEV = "${env.TAG}-${env.VERSION}-${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Update Deployment and Service specification') {
            steps {
                container('git') {
                    withCredentials([usernamePassword(credentialsId: 'git-creds-ace', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
                        sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/maas-hot"
                        //sh "cd k8s-deploy-staging/ && sed 's#value: \"DT_CUSTOM_PROP_PLACEHOLDER\".*#value: \"${env.DT_CUSTOM_PROP}\"#' ${env.APP_NAME}.yml > manifest-gen/${env.APP_NAME}.yml"
                        sh "cd maas-hot/ && sed -i 's#image: .*#image: ${env.TAG_STAGING}#' manifests/gen/${env.APP_NAME}.yml"
                        sh "cd maas-hot/ && git add manifests/gen/${env.APP_NAME}.yml && git commit -m 'Update ${env.APP_NAME} version ${env.VERSION}'"
                        sh "cd maas-hot/ && git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/maas-hot"
                        sh "rm -rf maas-hot"
                    }
                }
            }
        }
        stage('Deploy to staging namespace') {
            steps {
                checkout scm
                container('kubectl') {
                    sh "kubectl -n staging apply -f manifests/gen/${env.APP_NAME}.yml"
                }
            }
        }
    }
}