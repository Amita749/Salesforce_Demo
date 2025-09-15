#!groovy
import groovy.json.JsonSlurperClassic

node {
    def BUILD_NUMBER = env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR = "tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS'
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY

    // Explicitly set path to Salesforce CLI (Windows)
    def toolbelt = "\"C:\\sf\\bin\\sf.cmd\""

    stage('Checkout Source') {
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Authorize Org') {
            def rc
            if (isUnix()) {
                // Linux/Mac syntax
                rc = sh returnStatus: true, script: "${toolbelt} org login jwt --client-id ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwt-key-file ${jwt_key_file} --instance-url ${SFDC_HOST} --set-default-dev-hub"
            } else {
                // Windows syntax
                rc = bat returnStatus: true, script: "${toolbelt} org login jwt --client-id ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwt-key-file ${jwt_key_file} --instance-url ${SFDC_HOST} --set-default-dev-hub --loglevel debug"
            }

            if (rc != 0) {
                println 'Authorization failed'
                error 'Hub org authorization failed'
            } else {
                println 'Authorization successful'
            }
        }

        stage('Deploy Code') {
            def rmsg
            if (isUnix()) {
                rmsg = sh returnStdout: true, script: "${toolbelt} project deploy start --manifest manifest/package.xml --target-org ${HUB_ORG}"
            } else {
                rmsg = bat returnStdout: true, script: "${toolbelt} project deploy start --manifest manifest/package.xml --target-org ${HUB_ORG}"
            }

            println rmsg
        }
    }
}
