pipeline {
    agent {
      dockerfile true
    }
 
    environment {
        // Uncomment below for hybrid
        //APIGEE_SA_CREDS = credentials('apigee-service-account')
        //HOME = '.'
        //ORG = 'amer-cs-hybrid-demo2'

        APIGEE_SA_CREDS = credentials('apigee-ngsaas-service-account')
        HOME = '.'
        ORG = 'ssv-apigee-ngsaas-project2'
    }

    stages {
        stage('Clean') {
            steps {
              script{

                // Uncomment below for hybrid
                /*if (env.GIT_BRANCH == "master") {
                    env.APIGEE_PREFIX = ""
                    env.APIGEE_PROFILE = "test"
                } else if (env.GIT_BRANCH == "prod") {
                    env.APIGEE_PREFIX = ""
                    env.APIGEE_PROFILE = "prod"
                } else { //feature branches
                    env.APIGEE_PREFIX = "jenkins"
                    env.APIGEE_PROFILE = "test"
                }*/

                if (env.GIT_BRANCH == "master") {
                    env.APIGEE_PREFIX = ""
                    env.APIGEE_PROFILE = "ngsaas-test"
                } else if (env.GIT_BRANCH == "prod") {
                    env.APIGEE_PREFIX = ""
                    env.APIGEE_PROFILE = "ngsaas-prod"
                } else { //feature branches
                    env.APIGEE_PREFIX = ""
                    env.APIGEE_PROFILE = "ngsaas-dev"
                }
              }
              sh "mvn clean"
            }
        }
        stage('Install Dependencies') {
            steps {
              sh "npm install --silent --no-fund"
            }
        }
        stage('Static Code analysis') {
            steps {
              sh "npm run lint"
            }
        }
        stage('Unit Test and Coverage') {
            steps {
              sh "npm run unit-test"
            }
        }
        stage('Process Resources') {
            steps {
              sh "mvn -ntp process-resources -P${env.APIGEE_PROFILE} -Dorg=${env.ORG} -Ddeployment.suffix=${env.APIGEE_PREFIX} -Dcommit=${env.GIT_COMMIT} -Dbranch=${env.GIT_BRANCH} -Duser.name=jenkins"
            }
        }
        stage('Pre-deployment configuration') {
            steps { 
              sh "mvn -ntp apigee-config:targetservers -P${env.APIGEE_PROFILE} -Dorg=${env.ORG} -Ddeployment.suffix=${env.APIGEE_PREFIX} -Dfile=${APIGEE_SA_CREDS} -Dapigee.config.options=update"
            }
        }
        stage('Package proxy bundle') {
            steps { 
              sh "mvn -ntp apigee-enterprise:configure -P${env.APIGEE_PROFILE} -Dorg=${env.ORG} -Ddeployment.suffix=${env.APIGEE_PREFIX}"
            }
        }
        stage('Deploy proxy bundle') {
            steps {
              sh "mvn -ntp apigee-enterprise:deploy -P${env.APIGEE_PROFILE} -Dorg=${env.ORG} -Ddeployment.suffix=${env.APIGEE_PREFIX} -Dfile=${APIGEE_SA_CREDS}"
            }
        }
        stage('Post-deployment configuration') {
            steps { 
              sh "mvn -ntp apigee-config:apps -P${env.APIGEE_PROFILE} -Dorg=${env.ORG} -Ddeployment.suffix=${env.APIGEE_PREFIX} -Dfile=${APIGEE_SA_CREDS} -Dapigee.config.options=delete"
              sh "mvn -ntp apigee-config:apiproducts apigee-config:developers apigee-config:apps -P${env.APIGEE_PROFILE} -Dorg=${env.ORG} -Ddeployment.suffix=${env.APIGEE_PREFIX} -Dfile=${APIGEE_SA_CREDS} -Dapigee.config.options=update"
            }
        }
        stage('Functional Test') {
          steps {
            sh "npm run integration-test"
          }
        }
    }

    post { 
        always {
            publishHTML(target: [
                                  allowMissing: false,
                                  alwaysLinkToLastBuild: false,
                                  keepAll: false,
                                  reportDir: "coverage",
                                  reportFiles: 'index.html',
                                  reportName: 'HTML Report'
                                ]
                        )

            step([
                  $class: 'CucumberReportPublisher',
                  fileIncludePattern: '**/reports.json',
                  jsonReportDirectory: "target",
                  sortingMethod: 'ALPHABETICAL',
                  trendsLimit: 10
                ])
        }
    }
}