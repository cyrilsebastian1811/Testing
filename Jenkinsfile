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

        // // 1st way
        // stage('Helm-Charts update') { 
        //     steps {
        //         sh "ls"
        //         sh "pwd"
        //         echo "${BUILD_NUMBER}"
        //         sh "helm package ./webapp-backend --version 0.1.${BUILD_NUMBER} -u"
        //         sh "ls -l"
        //         sh "tar xvf webapp-backend-0.1.${BUILD_NUMBER}.tgz webapp-backend/Chart.yaml"
        //         sh "rm *.tgz"
        //         sh "ls -l"
        //         sh "git checkout test" 
        //         sh "git branch"
        //         sh "git commit -am 'version upgrade to 0.1.${BUILD_NUMBER} by jenkins'"
        //         sshagent (credentials: ['github-ssh']) {
        //             sh("git push origin test")
        //         }
        //     }
        // }

        // 2nd way
        stage('Helm-Charts update') { 
            steps {
                sh "ls"
                sh "pwd"
                echo "${BUILD_NUMBER}"
                sh "git checkout test"
                sh "git branch"
                sh "sudo cat webapp-backend/Chart.yaml | yq r - version"
                sh "sudo yq write -i ./webapp-backend/Chart.yaml 'version' 0.1.${BUILD_NUMBER}"
                sh "sudo cat webapp-backend/Chart.yaml | yq r - version"
                sh "sudo yq write -i ./webapp-backend/values.yaml 'dockerImage' ${DOCKERHUB_CREDENTIALS_USR}/testing:${git_hash}"
                sh "sudo yq write -i ./webapp-backend/values.yaml 'imageCredentials.registry' https://index.docker.io/v1/"
                sh "git commit -am 'version upgrade to 0.1.${BUILD_NUMBER} by jenkins'"
                sshagent (credentials: ['github-ssh']) {
                    sh("git push origin test")
                }
            }
        }
    }
}