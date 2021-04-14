@Library('defra-library@v-9') _

import uk.gov.defra.ffc.Version

def pr = ''
def repoName = ''
String defaultBranch = 'main'

node {
  try {
    stage('Ensure clean workspace') {
        deleteDir()
      }

      stage('Checkout source code') {
        build.checkoutSourceCode(defaultBranch)
      }

    stage('Set PR and version variables') {
      (repoName, pr) = build.getVariables('', defaultBranch)
    }

    if (pr != '') {
        stage('Helm install') {
          helm.deployChart(environment, DOCKER_REGISTRY, repoName, tag, pr)
        }
      } else {
        stage('Publish chart') {
          helm.publishChart(DOCKER_REGISTRY, repoName, tag, HELM_CHART_REPO_TYPE)
        }

        stage('Trigger GitHub release') {
          withCredentials([
            string(credentialsId: 'github-auth-token', variable: 'gitToken')
          ]) {
            String commitMessage = utils.getCommitMessage()
            release.trigger(tag, repoName, commitMessage, gitToken)
          }
        }

        stage('Trigger Deployment') {
          if (utils.checkCredentialsExist("$repoName-deploy-token")) {            
            withCredentials([
              string(credentialsId: "$repoName-deploy-token", variable: 'jenkinsToken')
            ]) {
              deploy.trigger(JENKINS_DEPLOY_SITE_ROOT, repoName, jenkinsToken, ['chartVersion': tag, 'environment': environment, 'helmChartRepoType': HELM_CHART_REPO_TYPE])
            }
          } else {            
            withCredentials([
              string(credentialsId: 'default-deploy-token', variable: 'jenkinsToken')
            ]) {
              deploy.trigger(JENKINS_DEPLOY_SITE_ROOT, repoName, jenkinsToken, ['chartVersion': tag, 'environment': environment, 'helmChartRepoType': HELM_CHART_REPO_TYPE])
            }
          }
        }
      }
    }

  } catch(e) {
    notifySlack.buildFailure(e.message, "#generalbuildfailures")
    throw e
  }
}