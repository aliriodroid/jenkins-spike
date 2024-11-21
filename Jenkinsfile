pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/sdk:6.0'
            args '-u root'
        }
    }
    
    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        DOTNET_SKIP_FIRST_TIME_EXPERIENCE = '1'
        PROJECT_PATH = './MiProyecto'
        TEST_PATH = './MiProyecto.Tests'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install .NET') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y wget curl
                    wget https://packages.microsoft.com/config/debian/11/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
                    dpkg -i packages-microsoft-prod.deb
                    rm packages-microsoft-prod.deb
                    apt-get update
                    apt-get install -y dotnet-sdk-6.0
                '''
            }
        }
        
        stage('Restore') {
            steps {
                sh "dotnet restore ${PROJECT_PATH}"
            }
        }
        
        stage('Build') {
            steps {
                sh "dotnet build ${PROJECT_PATH} -c Release"
            }
        }
        
        stage('Test') {
            steps {
                sh "dotnet test ${TEST_PATH}"
            }
        }
    }
}