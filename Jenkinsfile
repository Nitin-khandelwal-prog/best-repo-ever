pipeline {
agent any

environment {
// Salesforce environment variables
SF_CONSUMER_KEY = "${env.SF_CONSUMER_KEY}" // store in Jenkins as Secret Text
SF_USERNAME = "${env.SF_USERNAME}" // store in Jenkins as Secret Text
SF_INSTANCE_URL = "https://login.salesforce.com" // or test.salesforce.com for sandbox
PATH = "/usr/local/bin:/opt/homebrew/bin:$PATH"
}

stages {

stage('Checkout Source') {
steps {
checkout scm
}

}

stage('Authorize DevHub') {
steps {
withCredentials([file(credentialsId: 'CICD_PRIVATE_KEY', variable: 'server_key_file')]) {
sh """
sfdx auth:jwt:grant \
--clientid ${SF_CONSUMER_KEY} \
--jwtkeyfile ${server_key_file} \
--username ${SF_USERNAME} \
--instanceurl ${SF_INSTANCE_URL} \
--setdefaultdevhubusername
"""
}
}
}

stage('Create Scratch Org') {
steps {
sh """
 'sfdx force:org:create -f my-sfdx-project/config/project-scratch-def.json -a ciorg -d 7'
"""
}
}

stage('Push Source To Scratch Org') {
steps {

sh 'sfdx project deploy start --target-org ciorg'
}
}

stage('Run Apex Tests') {
steps {
sh 'sfdx apex run test --target-org ciorg --wait 10 --result-format human --code-coverage'
}
}

stage('Delete Scratch Org') {
steps {
sh 'sfdx org delete scratch --target-org ciorg --no-prompt'
}
}
}
}
