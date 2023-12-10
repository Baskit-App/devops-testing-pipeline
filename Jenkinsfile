
def notificationDevBuild() {
    if(currentBuild.currentResult == 'SUCCESS'){
        def message = "@here Build <${env.BUILD_URL}|${currentBuild.displayName}> " + "${currentBuild.currentResult} Ready To Deploy In Testing Environment"
        slackSend(message: message, channel: "#tech-jenkins-pipeline-cicd", token: "slack-token", color: 'good')
    }else {
        def message = "@here Build <${env.BUILD_URL}|${currentBuild.displayName}> " + "${currentBuild.currentResult} Solving The Issue and Rebuild Again"
        slackSend(message: message ,channel: "#tech-jenkins-pipeline-cicd", token: "slack-token", color: 'danger')
    }
}

def notificationDevDeploy() {
    if(currentBuild.currentResult == 'SUCCESS'){
        def message = "@here Build <${env.BUILD_URL}|${currentBuild.displayName}> " + "${currentBuild.currentResult} Already Done Deployment In Testing Environment"
        slackSend(message: message, channel: "#tech-jenkins-pipeline-cicd", token: "slack-token", color: 'good')
    }else {
        def message = "@here Build <${env.BUILD_URL}|${currentBuild.displayName}> " + "${currentBuild.currentResult} Solving The Issue and Redeploy again"
        slackSend(message: message ,channel: "#tech-jenkins-pipeline-cicd", token: "slack-token", color: 'danger')
    }
}


def notifyBuild(String buildStatus = 'STARTED', String colorCode = '#5492f7', String notify = '') {

  def channel = "#tech-jenkins-pipeline-cicd"
  def token   = "slack-token"

  def commit = sh(returnStdout: true, script: 'git log -n 1 --format="%H"').trim()
  def author = sh(returnStdout: true, script: "git --no-pager show -s --format='%an'").trim()
  def link = "Trigger By: ${author}| ${commit}"
  def shortCommit = commit.take(6)
  def title = sh(returnStdout: true, script: 'git log -n 1 --format="%s"').trim()
  def subject = "<${link}|${shortCommit}> ${title}"

  def summary = "${buildStatus}\n Job URL: <${env.BUILD_URL}| ${env.JOB_NAME} [${env.BUILD_NUMBER}]>\n${subject}\n${notify}"

  slackSend (channel: "#${channel}", color: colorCode, message: summary, token: "slack-token")

}


pipeline {
    agent {
        label 'dev-beta'
    }
    environment {
        REPO_BRANCH = "develop"
        REPO_URL    = "https://github.com/Baskit-App/devops-testing-pipeline.git"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                          branches: [[name: "${env.REPO_BRANCH}"]], 
                          doGenerateSubmoduleConfigurations: false, 
                          extensions: [], 
                          submoduleCfg: [], 
                          userRemoteConfigs: [[
                              credentialsId: 'github-jenkins', 
                              url: "${env.REPO_URL}"
                        ]]])
                script {
                    def last_git = sh (
                        script: 'git for-each-ref --sort=committerdate refs/remotes/',
                        returnStdout: true
                    ).trim()
                    do_build = last_git.endsWith("${env.REPO_BRANCH}")
                }
            }
        }  
        
        
        stage ('Deploy TO Testing Environment'){
            steps {   
                notifyBuild()
                echo "checkout ${env.REPO_BRANCH}"
                checkout([$class: 'GitSCM', 
                          branches: [[name: "${env.REPO_BRANCH}"]], 
                          doGenerateSubmoduleConfigurations: false, 
                          extensions: [], 
                          submoduleCfg: [], 
                          userRemoteConfigs: [[
                              credentialsId: 'github-jenkins', 
                              url: "${env.REPO_URL}"
                        ]]])
                echo "testing"

                   
            }
        }
        
    }
    post{
        always{
            notificationDevDeploy()
        }
    }
}