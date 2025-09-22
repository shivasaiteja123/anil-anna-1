pipeline {
    agent any
    environment {
        sourceFolder = "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\ai\\Publish\\" 
        destinationFolder = "E:\\aiscipro-demo\\test\\ai"
        sonarScannerPath = "C:\sonar-scanner-cli-7.0.2.4839-windows-x64\sonar-scanner\bin"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                          branches: [[name: '*/main']], 
                          userRemoteConfigs: [[credentialsId: 'CORE', 
                                               url: 'https://github.com/Coresonantiot/AiSciPro-AI-Hexa.git']]])
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        bat "\"${sonarScannerPath}\" -Dsonar.projectKey=SonarQube -Dsonar.sources=. -Dsonar.host.url=http://13.233.108.235:9000 -Dsonar.login=sqa_294a4cbeeac09dec243f57f10da8f4cb0c3a0241"
                    }
                }
            }
        }

        stage('Restore All Project Dependencies') {
            steps {
                echo 'Restore AiSciPro-AI-Hexa Project Dependencies'
                bat "dotnet restore .\\src\\AiSciPro.Core\\AiSciPro.AI.Service.sln"
            }
        }

        stage('Build AiSciPro-AI-Hexa Project and Microservices') {
            steps {
                echo 'Building AiSciPro-AI-Hexa Project and Microservices'
                bat "dotnet build .\\src\\AiSciPro.Core\\AiSciPro.AI.Service.sln"
            }
        }

        stage('Publish AiSciPro-AI-Hexa Project and Microservices') {
            steps {
                echo 'Publishing AiSciPro.AI.Gateway'
                bat "dotnet publish .\\src\\AiSciPro.Core\\AiSciPro.Gateway\\AiSciPro.Gateway.ProjectManagement\\AiSciPro.Gateway.ProjectManagement.csproj -o Publish\\AiSciPro.Gateway.ProjectManagement"
            }
        }

        stage('Deployment Configuration') {
            steps {
                echo 'Configuring Deployment Config for AiSciPro-AI-Hexa Azure Cloud'
                echo 'Copy Deploy Config AiSciPro.AI.Gateway.ProjectManagement'
                bat "copy /Y .\\DeployConfig\\DEV\\AiSciPro.Gateway.ProjectManagement\\. .\\Publish\\AiSciPro.Gateway.ProjectManagement\\"
                echo 'All Deploy Configuration Set Successfully'
            }
        }

        stage('Archive') {
            steps {
                echo 'Archiving AiSciPro-AI-Hexa Artifacts'
                archiveArtifacts artifacts: 'Publish/**', onlyIfSuccessful: true
            }
        }

        stage('Deploy - Staging') {
            steps {
                script {
                    echo 'Preparing AiSciPro-AI-Hexa Publish Artifacts for Deploying on Local server.'
                    bat "xcopy /Y \"${sourceFolder}\" \"${destinationFolder}\" /E /I"
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning Up Workspace'
            bat "rmdir /q /s .\\Publish"
        }
        success {
            echo 'CICD QA AiSciPro-AI-Hexa Succeeded!'
        }
        unstable {
            echo 'Something Wrong CICD Failed'
        }
        failure {
            echo 'CICD Failed'
        }
    }
}
