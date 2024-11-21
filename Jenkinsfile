pipeline {
    agent any
    
    environment {
        DOTNET_SDK_VERSION = '6.0'
        PROJECT_PATH = './MiProyecto'
        TEST_PATH = './MiProyecto.Tests'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
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
        
        stage('Publish') {
            steps {
                sh "dotnet publish ${PROJECT_PATH} -c Release -o ./publish"
            }
        }
    }
    
    post {
        success {
            archiveArtifacts artifacts: 'publish/**', fingerprint: true
        }
        
        failure {
            echo 'Build or tests failed'
        }
    }
}