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
                sh "dotnetRestore ${PROJECT_PATH}"
            }
        }
        
        stage('Build') {
            steps {
                sh "dotnetBuild ${PROJECT_PATH} -c Release"
            }
        }
        
        stage('Test') {
            steps {
                sh "dotnetTest ${TEST_PATH}"
            }
        }
        
        stage('Publish') {
            steps {
                sh "dotnetPublish ${PROJECT_PATH} -c Release -o ./publish"
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