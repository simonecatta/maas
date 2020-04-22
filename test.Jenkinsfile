@Library('ace@master') _ 

pipeline {
    parameters {
        string(name: 'APP_NAME', defaultValue: 'simplenodeservice', description: 'The name of the service to deploy.', trim: true)
    }
    agent {
        label 'kubegit'
    }
    stages {
        stage('Run production ready e2e check in staging') {
            steps {
                checkout scm
                container('jmeter') {
                    script {
                    def status = executeJMeter ( 
                        scriptName: "jmeter/simplenodeservice_load.jmx",
                        resultsDir: "e2eCheck_${env.APP_NAME}_staging_${BUILD_NUMBER}",
                        serverUrl: "simplenode.staging", 
                        serverPort: 80,
                        checkPath: '/health',
                        vuCount: 10,
                        loopCount: 1000,
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

        stage('Manual approval') {
            // no agent, so executors are not used up when waiting for approvals
            agent none
            steps {
                script {
                    try {
                        timeout(time:10, unit:'MINUTES') {
                            env.APPROVE_PROD = input message: 'Promote to Production', ok: 'Continue',
                                parameters: [choice(name: 'APPROVE_PROD', choices: 'YES\nNO', description: 'Deploy from STAGING to PRODUCTION?')]
                            if (env.APPROVE_PROD == 'YES'){
                                env.DPROD = true
                            } else {
                                env.DPROD = false
                            }
                        }
                    } catch (error) {
                        env.DPROD = true
                        echo 'Timeout has been reached! Deploy to PRODUCTION automatically activated'
                    }
                }
            }
        }

        stage('Promote to production') {
            when {
                expression {
                    return env.APPROVE_PROD == 'YES'
                }
            }
            steps {
                build job: "4. Deploy production",
                parameters: [
                    string(name: 'APP_NAME', value: "${env.APP_NAME}")
                ]
            }
        }  
    }
}