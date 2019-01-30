#!groovy
import groovy.json.JsonSlurperClassic
node {
  def BUILD_NUMBER=env.BUILD_NUMBER
  def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
  def SFDC_USERNAME

  def HUB_ORG=env.HUB_ORG_DH
  def SFDC_HOST = env.SFDC_HOST_DH
  def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
  def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
  def ORG_ALIAS=env.ORG_ALIAS

  println 'KEY IS'
  println JWT_KEY_CRED_ID
  println HUB_ORG
  println SFDC_HOST
  println CONNECTED_APP_CONSUMER_KEY
  def toolbelt = tool 'toolbelt'

  stage('checkout source') {
    // When running in multi-branch job, one must issue this command
    checkout scm
  }

  withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
    stage('Deploye Code') {
      if (isUnix()) {
        rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
      } else {
        rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
      }
      println rc

      if (rc != 0) {
        error 'Dev Hub Org authorization failed!'
      } else {
        // Create scratch org
        if (isUnix()) {
          rc = sh returnStdout: true, script: "${toolbelt} force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername -a ${ORG_ALIAS}"
        } else {
          rc = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername -a ${ORG_ALIAS}"
        }
        println rc

        // Push source to scratch org
        if (isUnix()) {
          rc = sh returnStdout: true, script: "${toolbelt} force:source:push"
        } else {
          rc = bat returnStdout: true, script: "\"${toolbelt}\" force:source:push"
        }
        println rc

        // Run unit tests on the scratch org
        if (isUnix()) {
          rc = sh returnStdout: true, script: "${toolbelt} force:apex:test:run --testlevel RunLocalTests --resultformat tap --targetusername ${ORG_ALIAS} --synchronous"
        } else {
          rc = bat returnStdout: true, script: "\"${toolbelt}\" force:apex:test:run --testlevel RunLocalTests --resultformat tap --targetusername ${ORG_ALIAS} --synchronous"
        }
        println rc

        // Delete the scratch org.
        if (isUnix()) {
          rc = sh returnStdout: true, script: "${toolbelt} force:org:delete -p -u ${ORG_ALIAS}"
        } else {
          rc = bat returnStdout: true, script: "\"${toolbelt}\" force:org:delete -p -u ${ORG_ALIAS}"
        }
        println rc
      }
    }
  }
}
