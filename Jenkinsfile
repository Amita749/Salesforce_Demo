#!groovy
node {
    def sfcli = "\"C:\\sf\\bin\\sf.cmd\""

    // Dev Hub credentials from Jenkins environment variables
    def HUB_ORG = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH

    // Scratch org alias
    def SCRATCH_ORG_ALIAS = "ScratchOrg1"

    stage('Checkout Source') {
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_file')]) {

        stage('Authorize Dev Hub') {
            def rc = bat returnStatus: true, script: "${sfcli} org login jwt --client-id ${CONNECTED_APP_KEY} --username ${HUB_ORG} --jwt-key-file ${jwt_file} --instance-url ${SFDC_HOST} --set-default-dev-hub --loglevel debug"
            if (rc != 0) {
                error '❌ Dev Hub authorization failed'
            }
            echo '✅ Dev Hub authorized successfully'
        }

        stage('Create Scratch Org') {
            def rc = bat returnStatus: true, script: "${sfcli} org create scratch --definition-file config/dev-scratch-def.json --set-default --alias ${SCRATCH_ORG_ALIAS} --duration-days 7"
            if (rc != 0) {
                error "❌ Failed to create scratch org '${SCRATCH_ORG_ALIAS}'"
            }
            echo "✅ Scratch org '${SCRATCH_ORG_ALIAS}' created"
        }

        stage('Deploy to Scratch Org') {
            def rc = bat returnStatus: true, script: "${sfcli} project deploy start --manifest manifest/package.xml --target-org ${SCRATCH_ORG_ALIAS}"
            if (rc != 0) {
                error "❌ Deployment failed"
            }
            echo "✅ Deployment successful to '${SCRATCH_ORG_ALIAS}'"
        }

        

       
    }
}

