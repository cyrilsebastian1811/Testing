// Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
        git_hash = null
        image = null
        // DOCKERHUB_CREDENTIALS_USR and DOCKERHUB_CREDENTIALS_PSW automatically available
    }
    stages {
        stage('Checkout Testing') { 
            steps {
                script {
                    git_info = checkout([
                        $class: 'GitSCM', branches: [[name: '*/a1']], 
                        doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [],
                        userRemoteConfigs: [[
                            credentialsId: 'github-ssh', 
                            url: 'git@github.com:cyrilsebastian1811/Testing.git'
                        ]]
                    ])
                    git_hash = "${git_info.GIT_COMMIT[0..6]}"
                }

                echo "${git_hash}"
            }
        }
        stage('Build Image') { 
            steps {
                script {
                    image = docker.build("${DOCKERHUB_CREDENTIALS_USR}/testing")
                }
            }
        }
        stage('Push Image') { 
            steps {
                script {
                    def docker_info = docker.withRegistry("https://registry.hub.docker.com/", "dockerhub_credentials") {
                        image.push("${git_hash}")
                    }
                }
            }
        }
        stage('Remove Images') { 
            steps {
                sh "docker system prune --all -f"
            }
        }

        stage('Checkout Helm-Charts') { 
            steps {
                script {
                    git_info = checkout([
                        $class: 'GitSCM', branches: [[name: '*/test']], 
                        doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [],
                        userRemoteConfigs: [[
                            credentialsId: 'github-ssh', 
                            url: 'git@github.com:cyrilsebastian1811/helm-charts.git'
                        ]]
                    ])
                }
            }
        }

        stage('Helm-Charts update') { 
            steps {
                sh "ls"
                sh "cd webapp-backend"
                sh "helm upgrade --set version=0.1.${BUILD_NUMBER} backend ."
                sh "cat Chart.yaml"
                echo "${BUILD_NUMBER}"
            }
        }
    }
}