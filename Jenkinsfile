import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv([
    'AZURE_SUBSCRIPTION_ID=4db130d7-9c6a-4238-be5e-9ecbd6ba2075',
    'AZURE_TENANT_ID=672335e9-2aa4-4282-8542-4371bbd9ef19'
  ]) {

    stage('init') {
      checkout scm
    }

    stage('build') {
      sh 'mvn clean package'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'yuzewebapptest123'

      // 登录 Azure
      withCredentials([usernamePassword(
        credentialsId: 'AzureServicePrincipal',
        usernameVariable: 'AZURE_CLIENT_ID',
        passwordVariable: 'AZURE_CLIENT_SECRET'
      )]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      // 获取 FTP 发布凭据
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson

      // 上传 WAR 包
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"

      // 退出 Azure 登录
      sh 'az logout'

      echo "Deployment to $webAppName completed successfully."
    }
  }
}
