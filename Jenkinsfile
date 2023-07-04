import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

pipeline {
  agent any
  
  environment {
    AZURE_SUBSCRIPTION_ID = '<b18faa87-0714-43b3-bb71-b49bff521fbb>'
    AZURE_TENANT_ID = '<59bdeeee-d88d-49ad-a8c4-94c771cfb354>'
  }
  
  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }
    
    stage('build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('deploy') {
      steps {
        script {
          def resourceGroup = '<firstwebapp2899_group>'
          def webAppName = '<nikhithaapp>'
          
          // login Azure
withCredentials([azureServicePrincipal('ak')]) {
    sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
}
          
          // get publish settings
          def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
          def ftpProfile = getFtpPublishProfile(pubProfilesJson)
          
          // upload package
          sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
          
          // log out
          sh 'az logout'
        }
      }
    }
  }
}
