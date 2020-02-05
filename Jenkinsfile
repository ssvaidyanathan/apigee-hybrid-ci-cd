pipeline {
    agent {
      dockerfile true
    }

    environment {
        APIGEE_CREDS = credentials('apigee')
        APIGEE_SA_CREDS = credentials('apigee-service-account')
        HOME = '.'
    }

    stages {
        stage('Clean') {
            steps {
              script{
                if (env.GIT_BRANCH == "master") {
                    env.APIGEE_PREFIX = ""
                    env.APIGEE_PROFILE = "test"
                } else if (env.GIT_BRANCH == "prod") {
                    env.APIGEE_PREFIX = ""
                    env.APIGEE_PROFILE = "prod"
                } else { //feature branches
                    env.APIGEE_PREFIX = "jenkins"
                    env.APIGEE_PROFILE = "test"
                }
              }
              sh "echo ${APIGEE_SA_CREDS}"
              sh "mvn clean"
            }
        }
        stage('Static Code Analysis, Unit Test and Coverage') {
            steps {
              sh "mvn -ntp test -P${env.APIGEE_PROFILE} -Ddeployment.suffix=${env.APIGEE_PREFIX} -Dcommit=${env.GIT_COMMIT} -Dbranch=${env.GIT_BRANCH} -Duser.name=jenkins"
            }
        }
        stage('Package proxy bundle') {
            steps { 
              sh "mvn -ntp apigee-enterprise:configure -P${env.APIGEE_PROFILE} -Ddeployment.suffix=${env.APIGEE_PREFIX}"
            }
        }
        stage('Deploy proxy bundle') {
            steps {
              sh "mvn -ntp apigee-enterprise:deploy -P${env.APIGEE_PROFILE} -Ddeployment.suffix=${env.APIGEE_PREFIX} -Dusername=${APIGEE_CREDS_USR} -Dpassword=${APIGEE_CREDS_PSW}"
            }
        }
        stage('Functional Test') {
          steps {
            sh "node ./node_modules/cucumber/bin/cucumber.js target/test/integration/features --format json:target/reports.json"
          }
        }
    }

    post { 
        always {
            publishHTML(target: [
                                  allowMissing: false,
                                  alwaysLinkToLastBuild: false,
                                  keepAll: false,
                                  reportDir: "coverage/lcov-report",
                                  reportFiles: 'index.html',
                                  reportName: 'HTML Report'
                                ]
                        )

            step([
                    $class: 'CucumberReportPublisher',
                    fileExcludePattern: '',
                    fileIncludePattern: "**/reports.json",
                    ignoreFailedTests: false,
                    jenkinsBasePath: '',
                    jsonReportDirectory: "target",
                    missingFails: false,
                    parallelTesting: false,
                    pendingFails: false,
                    skippedFails: false,
                    undefinedFails: false
                    ])
        }
    }
}