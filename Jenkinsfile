@Library('Jenkins_shared_library') _

def COLOR_MAP = [
    'FAILURE' : 'danger',
    'SUCCESS' : 'good'
]
pipeline{
    agent any
    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Select create or destroy.')
    }
    stages{
        stage('clean workspace'){
            steps{
                cleanWorkspace()
            }
        }
        stage('checkout from Git'){
            steps{
                checkoutGit('https://github.com/mukeshr-29/Youtube-project-12.git', 'main')
            }
        }
    }
    post {
         always {
            script{
                echo 'Slack Notifications'
                slackSend (
                    channel: '#jenkins-youtube',   #change your channel name
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
               )
           }
       }
   }
}