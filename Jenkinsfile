@Library('Jenkins_shared_library') _
def configUtils = loadConfig()

pipeline {
  agent any
  parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Select create or destroy.')
        string(name: 'DOCKER_HUB_USERNAME', defaultValue: 'nagarjunacse02', description: 'Docker Hub Username')
        string(name: 'IMAGE_NAME', defaultValue: 'favPlayList', description: 'Docker Image Name')
        string(name: 'GITHUB_USERNAME', description: 'Github User Name', defaultValue: 'nagarjunacse02')
    }
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        URL_WEBHOOK = credentials("URL_WEBHOOK")
    }
  stages {
    stage ('clean workspace') {
      steps {
        cleanWorkspace()
      }
    }
    stage ('source code checkout') {
        steps { 
           checkoutGit('https://github.com/params.GITHUB_USERNAME/My-Favourite-Playlist.git', 'main') 
        }
    }
    stage('sonarqube Analysis'){
    when { expression { params.action == 'create'}}    
        steps{
            sonarAnalysis()
        }
    }
    stage('sonarqube QualitGate'){
    when { expression { params.action == 'create'}}    
        steps{
            script{
                def credentialsId = 'sonar-token'
                qualityGate(credentialsId)
            }
        }
    }
    stage('Npm install'){
    when { expression { params.action == 'create'}}    
        steps{
            npmInstall()
        }
    }
    stage('Trivy FS Scan') { 
    when { expression { params.action == 'create'}}
        steps { 
            trivyFileScan()
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
  }
}
