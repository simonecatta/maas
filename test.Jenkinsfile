pipeline {
    agent {
        label 'nodejs'
    }
    environment {
        APP_NAME = "simplenodeservice"
    }
    stages {
        stage('Run production ready e2e check in staging') {
            steps {
                container('jmeter') {
                    script {
                    def status = executeJMeter ( 
                        scriptName: "jmeter/simplenodeservice_load.jmx",
                        resultsDir: "e2eCheck_${env.APP_NAME}_staging_${BUILD_NUMBER}",
                        serverUrl: "simplenode.staging", 
                        serverPort: 80,
                        checkPath: '/health',
                        vuCount: 1,
                        loopCount: 1,
                        LTN: "e2eCheck_${BUILD_NUMBER}",
                        funcValidation: false,
                        avgRtValidation: 4000
                    )
                    if (status != 0) {
                        currentBuild.result = 'FAILED'
                        error "Production ready e2e check in staging failed."
                    }
                    }
                }
            }
        }
    }
}