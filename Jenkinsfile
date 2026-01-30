pipeline {
  agent any
  triggers {
    githubPush()
    pollSCM('H H * * *')
  }
  environment {
    MAIL_RECIPIENTS = 'dev+tests-reports@wazo.community'
  }
  options {
    skipStagesAfterUnstable()
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }
  stages {
    stage('Debian build and deploy') {
      steps {
        build job: 'build-package-no-arch', parameters: [
          string(name: 'PACKAGE', value: "${JOB_NAME}"),
          string(name: 'DISTRIBUTION', value: "wazo-dev-tools"),
        ]
      }
    }
    // Still used by build-package-* jobs
    stage ('Deploy on Jenkins') {
      steps {
        sh '''
          sudo apt-get update
          sudo apt-get install --yes wazo-ci
        '''
      }
    }
    stage ('Deploy Jenkins Private') {
      steps {
        withCredentials([usernamePassword(
            credentialsId: 'jenkins-private-infra-bot-token',
            usernameVariable: 'JENKINS_USER',
            passwordVariable: 'API_TOKEN'
        )]) {
          script {
            sh 'curl -X POST --fail --user ${JENKINS_USER}:${API_TOKEN} https://jenkins.wazo.io/job/deploy-wazo-ci/build'
          }
        }
      }
    }
  }
  post {
    failure {
      emailext to: "${MAIL_RECIPIENTS}", subject: '${DEFAULT_SUBJECT}', body: '${DEFAULT_CONTENT}'
    }
    fixed {
      emailext to: "${MAIL_RECIPIENTS}", subject: '${DEFAULT_SUBJECT}', body: '${DEFAULT_CONTENT}'
    }
  }
}
