@Library('Jenkins_shared_library') _

def COLOR_MAP = [
    'FAILURE' : 'danger',
    'SUCCESS' : 'good'
]
pipeline{
    agent any
    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Select create or destroy.')
        string(name: 'DOCKER_HUB_USERNAME', defaultValue: 'mukeshr29', description: 'Docker Hub Username')
        string(name: 'IMAGE_NAME', defaultValue: 'youtube-project', description: 'Docker Image Name')
    }
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonarqube'
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
        stage('sonarqube Analysis'){
        when { expression { params.action == 'create'}}    
            steps{
                sonarqubeAnalysis()
            }
        }
        stage('sonarqube QualitGate'){
        when { expression { params.action == 'create'}}    
            steps{
                script{
                    def credentialsId = 'sonarqube'
                    qualityGate(credentialsId)
                }
            }
        }
        stage('Npm'){
        when { expression { params.action == 'create'}}    
            steps{
                npmInstall()
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('trivy scan'){
            steps{
                trivyFs()
            }
        }
        stage('Docker Build'){
        when { expression { params.action == 'create'}}    
            steps{
                script{
                   def dockerHubUsername = params.DOCKER_HUB_USERNAME
                   def imageName = params.IMAGE_NAME

                   dockerBuild(dockerHubUsername, imageName)
                }
            }
        }
        stage('Trivy iamge'){
        when { expression { params.action == 'create'}}    
            steps{
                trivyImage()
            }
        }
        stage('Run container'){
        when { expression { params.action == 'create'}}    
            steps{
                runContainer()
            }
        }
        stage('Remove container'){
        when { expression { params.action == 'delete'}}    
            steps{
                deleteContainer()
            }
        }
        stage('kube deploy'){
            when { expression { params.action == 'create'}}    
            steps{
                kubeDeploy()
            }
        }
        stage('kube deleter'){
        when { expression { params.action == 'delete'}}    
            steps{
                kubeDelete()
            }
        }
    }
    post {
         always {
            script{
                echo 'Slack Notifications'
                slackSend (
                    channel: '#jenkins-youtube',
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
               )
           }
       }
   }
}