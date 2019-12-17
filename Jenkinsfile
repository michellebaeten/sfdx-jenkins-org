#!groovy
import groovy.json.JsonSlurperClassic

node {

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID
    def DEPLOYDIR='force-app'
    def TEST_LEVEL='RunLocalTests'
    


    def toolbelt = tool 'toolbelt'


    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }


    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------

    withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')]) {
        // -------------------------------------------------------------------------
        // Authenticate to Salesforce using the server key.
        // -------------------------------------------------------------------------


        stage('Authorize to Salesforce') {
//	    rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:auth:jwt:grant --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername --instanceurl https://login.salesforce.com"		
       		
            rc = command "${toolbelt} force:auth:jwt:grant --instanceurl https://login.salesforce.com --clientid ${SF_CONSUMER_KEY} --setdefaultdevhubusername --jwtkeyfile server.key --username ${SF_USERNAME}"
            if (rc != 0) {
                error 'Salesforce org authorization failed.'
            }
            rmsg = bat returnStdout: true, script: "${toolbelt} force:org:create -s -f config/project-scratch-def.json -a dreamhouse-org2 -v ${SF_USERNAME}"
            //printf rmsg
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != "ok") { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.username
            robj = null
        
        }
        stage('Push Source') {
	     rc = command "${toolbelt} force:source:push --targetusername ${SF_USERNAME} "
            if (rc != 0) {
                error 'Salesforce push failed.'
            }
        }

        // -------------------------------------------------------------------------
        // Deploy metadata and execute unit tests.
        // -------------------------------------------------------------------------
       // stage('Deploy and Run Tests') {
	   //  rc = command "${toolbelt} force:mdapi:deploy --wait 10 --deploydir ${DEPLOYDIR} --targetusername UAT --testlevel ${TEST_LEVEL}"
        //    if (rc != 0) {
        //        error 'Salesforce deploy and test run failed.'
        //    }
       // }


        // -------------------------------------------------------------------------
        // Example shows how to run a check-only deploy.
        // -------------------------------------------------------------------------

        //stage('Check Only Deploy') {
        //    rc = command "${toolbelt}/sfdx force:mdapi:deploy --checkonly --wait 10 --deploydir ${DEPLOYDIR} --targetusername UAT --testlevel ${TEST_LEVEL}"
        //    if (rc != 0) {
        //        error 'Salesforce deploy failed.'
        //    }
        //}
    }
}

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
		return bat(returnStatus: true, script: script);
    }
}