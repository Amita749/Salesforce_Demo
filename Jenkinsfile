node {
    def sfcli = "\"C:\\sf\\bin\\sf.cmd\""

    // Environment variables from Jenkins
    def HUB_ORG = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH
    def SCRATCH_ORG_1_ALIAS = env.SCRATCH_ORG_1_ALIAS
    def SCRATCH_ORG_2_ALIAS = env.SCRATCH_ORG_2_ALIAS

    stage('Checkout Source') {
        checkout scm
    }

    // Function to get the username from a scratch org alias
    def getUsernameFromAlias(String alias) {
        def output = bat(returnStdout: true, script: "${sfcli} org display ${alias} --json").trim()
        def json = readJSON text: output
        return json.result.username
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_file')]) {

        stage('Authorize Dev Hub') {
            def rc = bat returnStatus: true, script: "${sfcli} org login jwt --client-id ${CONNECTED_APP_KEY} --username ${HUB_ORG} --jwt-key-file ${jwt_file} --instance-url ${SFDC_HOST} --set-default-dev-hub --loglevel debug"
            if (rc != 0) {
                error '❌ Dev Hub authorization failed'
            }
            echo '✅ Dev Hub authorized successfully'
        }

        stage("Deploy to ScratchOrg1") {
            def username1 = getUsernameFromAlias(SCRATCH_ORG_1_ALIAS)
            def rc = bat(returnStatus: true, script: "${sfcli} project deploy start --manifest manifest/package.xml --target-org ${username1}")
            if (rc != 0) {
                error "❌ Deployment to ${SCRATCH_ORG_1_ALIAS} failed"
            }
            echo "✅ Deployment to ${SCRATCH_ORG_1_ALIAS} succeeded"
        }

        stage("Deploy to ScratchOrg2") {
            def username2 = getUsernameFromAlias(SCRATCH_ORG_2_ALIAS)
            def rc = bat(returnStatus: true, script: "${sfcli} project deploy start --manifest manifest/package.xml --target-org ${username2}")
            if (rc != 0) {
                error "❌ Deployment to ${SCRATCH_ORG_2_ALIAS} failed"
            }
            echo "✅ Deployment to ${SCRATCH_ORG_2_ALIAS} succeeded"
        }
    }
}
