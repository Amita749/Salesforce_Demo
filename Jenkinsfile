#!groovy
node {
    def sfcli = "\"C:\\sf\\bin\\sf.cmd\""

    // Dev Hub credentials
    def HUB_ORG = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH

    // Scratch org aliases
    def SCRATCH_ORG_1 = 'ScratchOrg1'
    def SCRATCH_ORG_2 = 'ScratchOrg2'

    stage('Checkout Source') {
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_file')]) {

        stage('Authorize Dev Hub') {
            def rc = bat returnStatus: true, script: """
                ${sfcli} org login jwt ^
                  --client-id ${CONNECTED_APP_KEY} ^
                  --username ${HUB_ORG} ^
                  --jwt-key-file ${jwt_file} ^
                  --instance-url ${SFDC_HOST} ^
                  --set-default-dev-hub ^
                  --loglevel debug
            """
            if (rc != 0) { error 'Dev Hub authorization failed' }
            println '✅ Dev Hub authorized successfully'
        }

        stage("Deploy to ScratchOrg1") {
            def result = bat(returnStdout: true, script: """
                ${sfcli} project deploy start ^
                  --manifest manifest/package.xml ^
                  --target-org ${SCRATCH_ORG_1}
            """)
            println result
        }

        stage("Deploy to ScratchOrg2") {
            def result = bat(returnStdout: true, script: """
                ${sfcli} project deploy start ^
                  --manifest manifest/package.xml ^
                  --target-org ${SCRATCH_ORG_2}
            """)
            println result
        }
    }
}
