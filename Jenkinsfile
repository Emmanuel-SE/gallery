/* groovylint-disable CatchException, CompileStatic, DuplicateStringLiteral, LineLength, NestedBlockDepth, NoDef, SpaceAfterClosingBrace, VariableTypeRequired */
pipeline {
    agent any
    tools {
        nodejs 'nodeJs'
    }
    stages {
        stage('clone repository') {
            steps {
                git branch:'master', url:"${GIT_URL}"
            }
        }
        stage('Build project') {
            steps {
                sh 'npm install'
            }
        }
        stage('Tests') {
            steps {
                script {
                    def testResult = sh(script: 'npm run test', returnStatus: true)
                    if (testResult != 0) {
                        currentBuild.result = 'FAILURE'
                        FAILED_STAGE_NAME = "${STAGE_NAME}"
                        mail to: "${USER_EMAIL}", subject: "Test failed for build ${BUILD_ID}", body: "Test failed for build ${BUILD_ID}. Please check."
                    }else {
                        currentBuild.result = 'SUCCESS'
                        mail to: "${USER_EMAIL}", subject: "Test succesfull for build ${BUILD_ID}", body: "Test succesfull for build ${BUILD_ID}."
                    }
                }
            }
        }
        stage('Deploy to Heroku') {
            steps {
                script {
                    try {
                        withCredentials([usernameColonPassword(credentialsId: 'heroku', variable: 'HEROKU_CREDENTIALS')]) {
                    /* groovylint-disable-next-line GStringExpressionWithinString */
                            sh 'git push https://${HEROKU_CREDENTIALS}@git.heroku.com/gentle-waters-11368.git master'
                        }
                }catch (Exception e) {
                        FAILED_STAGE_NAME = "${STAGE_NAME}"
                    }
                }
            }
        }
    }
    post {
        always {
                script {
                    def slackMessage = "Pipeline ${currentBuild.result == 'FAILURE' ? 'failed' : 'succeeded'}"

                    if (currentBuild.result == 'FAILURE') {
                        slackMessage += " at stage: ${FAILED_STAGE_NAME}. Build ID: ${BUILD_ID}."
                    } else {
                        slackMessage += ". Build ID: ${BUILD_ID}. App URL:${APP_URL}"
                    }

                    slackSend channel: "${SLACK_CHANNEL}", color: currentBuild.result == 'FAILURE' ? 'danger' : 'good', iconEmoji: currentBuild.result == 'FAILURE' ? 'x' : 'thumbsup', message: slackMessage, tokenCredentialId: 'jenkins-slack'

                    if (currentBuild.result == 'FAILURE') {
                        slackSend channel: "${SLACK_CHANNEL}", color: 'danger', iconEmoji: 'x', message: 'The live site may be affected. Check the deployment.', tokenCredentialId: 'jenkins-slack'
                    }
                }
        }
    }
}
