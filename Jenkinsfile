 import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  return null
}

pipeline {
  agent any
     
  environment {
  (['AZURE_SUBSCRIPTION_ID=b18faa87-0714-43b3-bb71-b49bff521fbb',
           'AZURE_TENANT_ID=59bdeeee-d88d-49ad-a8c4-94c771cfb354']) }
 stages{
    stage('init') {
     step{
      checkout scm
     }
    }
 }

    stage('build') {
     step{
      sh 'mvn clean package'
    }
    }

    stage('deploy') {
     step{
      def resourceGroup = 'akgroup'
      def webAppName = 'workshopak'
      // Login to Azure
      withCredentials([usernamePassword(credentialsId: 'Azure1', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // Get publish settings
      def pubProfilesJson = sh(script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName --query '[].{publishMethod: publishMethod, publishUrl: publishUrl, userName: userName, userPWD: userPWD}' -o json", returnStdout: true).trim()
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)
      if (ftpProfile) {
        // Upload package
        sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
      } else {
        error "FTP publish profile not found"
      }
      // Log out
      sh 'az logout'
    }
    }
  }
}
}
