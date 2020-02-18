// Jenkinsfile (Declarative Pipeline)
pipeline {
    // agent any
    agent any
    options {
        skipDefaultCheckout(true)
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
        GIT_COMMIT = null
        git_info = null
        // DOCKERHUB_CREDENTIALS_USR and DOCKERHUB_CREDENTIALS_PSW automatically available
    }
    stages {
        stage('Environment variables') { 
            steps {
                echo "BUILD NUMBER: ${env.BUILD_NUMBER}"
                echo "${env}"
            }
        }
        stage('Checkout') { 
            checkout([
                $class: 'GitSCM', branches: [[name: '*/a1']], 
                doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [],
                    userRemoteConfigs: [[
                        credentialsId: 'github-ssh', 
                        url: 'git@github.com:cyrilsebastian1811/Testing.git'
                ]]
            ])
            steps {

            }
        }
        // stage('Build') { 
        //     steps {
        //         // 
        //     }
        // }
        // stage('Deploy') { 
        //     steps {
        //         // 
        //     }
        // }
    }
}