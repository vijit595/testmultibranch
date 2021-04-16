pipeline {
  agent any
  
  options {
      buildDiscarder(
        logRotator( 
            // number of build logs to keep
            numToKeepStr:'10',
            // history to keep in days
            daysToKeepStr: '30'
        )
    )
  }
  parameters {
    // Select the target org credentials for Product model deploy. Credentials need to be created before hand.
    choice(name: 'TARGET_ORG_CREDS', choices: ['DEV1', 'TEST1'], description: 'Select target org credentials.')
    booleanParam(name: 'CHECK_ONLY', defaultValue: true, description: 'Is check only deployment?')
  }
  environment {
    // vlocity configuration
    //TNS_ADMIN = "/opt/appl/jenkins/custom/jenkinsora"
    //ORACLE_HOME = "/opt/dbms/12c/Client_12R1"
    //PATH = "/opt/dbms/12c/Client_12R1/bin:/opt/data/jenkins/tools/node-v10.16.0-linux-x64/bin:/opt/data/jenkins/tools/kube:/sbin:/usr/sbin:/bin:/usr/bin"
    
    SF_LOGIN_URL = 'https://login.salesforce.com'
    TARGET_ORG_CREDS = credentials("${params.TARGET_ORG_CREDS}")
    TARGET_ENV = "${params.TARGET_ORG_CREDS}"
    SF_USERNAME = 'shalu@abc.com'//"${TARGET_ORG_CREDS_USR}"
    SF_PASSWORD = 'Sh@lu1603'//"${TARGET_ORG_CREDS_PSW}"
   
    SOURCE_DIR = "${WORKSPACE}/src"
    TARGET_DIR = "${WORKSPACE}/deploy"

    CHECK_ONLY = "${params.CHECK_ONLY}"
    GROOVY_LIBS = "${WORKSPACE}/packageBuilder/src/groovy"
    ANT_SALESFORCE_LIB = "${WORKSPACE}/build_scripts"
  }
  stages{
    
    
    stage('Generate Package.xml'){
      when {
        expression { env.DEPLOY_TYPE == 'SF' }
      }
      steps {
        script {
          packageBuilder = load "${GROOVY_LIBS}/package.groovy"
          packageBuilder.build(SOURCE_DIR, TARGET_DIR)
          sh "tree deploy"
        }
      }
    }
    stage('Deploy SF'){
      when {
        expression { env.DEPLOY_TYPE == 'SF' }
      }
      steps {
        script {
          ant = load "${GROOVY_LIBS}/ant.groovy"
          echo "Starting SF deployment to target org [${TARGET_ENV}]"
          echo "Getting [${ANT_SALESFORCE_LIB}/build.xml]"
          sh "cp ${ANT_SALESFORCE_LIB}/build.xml build.xml"
          ant.deploy(SF_LOGIN_URL, SF_USERNAME, SF_PASSWORD, TARGET_DIR, CHECK_ONLY )
        }
      }
    }
    
  }
  post { 
    failure { 
      script{
        notification = load "${GROOVY_LIBS}/notification.groovy"
        notification.sendEmailToDeveloper()
        //deleteDir()
      }
    }
    
  }
}

def getKindOfDeploy(){

  /*
  def DEPLOY_TYPE;
  if (BRANCH_NAME.contains('feature/sf/')){
    DEPLOY_TYPE = "SF"
  }else if(BRANCH_NAME.contains('feature/vlc/')){
    DEPLOY_TYPE = "VLC"
  }else if(BRANCH_NAME.contains('master')){
    DEPLOY_TYPE = "ALL"
  }
  
  env.DEPLOY_TYPE = DEPLOY_TYPE*/

  // Start with deploying everything (SF + Vlocity)
  env.DEPLOY_TYPE = "ALL"
}
