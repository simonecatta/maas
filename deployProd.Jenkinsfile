pipeline {
    parameters {
        string(name: 'APP_NAME', defaultValue: 'simplenodeservice', description: 'The name of the service to deploy.', trim: true)
    }
    agent {
        label 'kubegit'
    }
    stages {
        stage('Update production versions with latest versions from staging') {
            steps {
                container('kubectl') {
                    sh "sed -i \"s#image: .*#image: `kubectl -n staging get deployment -o jsonpath='{.items[*].spec.template.spec.containers[0].image}' --field-selector=metadata.name=${env.APP_NAME}`#\" manifests/production/${env.APP_NAME}.yml"
                    sh "manifests/production/${env.APP_NAME}.yml"
                    sh "kubectl -n production apply -f manifests/production/${env.APP_NAME}.yml"
                }
            }
        }
    }
}