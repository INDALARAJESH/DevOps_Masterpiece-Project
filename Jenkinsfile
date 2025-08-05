pipeline {
    agent any

    environment {
        NAME = "spring-app1"
        VERSION = "${env.BUILD_ID}"
        IMAGE_REPO = "sagar520"
        GIT_REPO_NAME = "DevOps_Masterpiece-Project"
        GIT_USER_NAME = "MAJJISAGAR"
    }

    tools { 
        maven 'maven-3.8.6' 
    }

    stages {
        stage('Checkout git') {
            steps {
                git branch: 'main', url: 'https://github.com/MAJJISAGAR/DevOps_Masterpiece-Project.git'
            }
        }

        stage('Get Git Commit Hash') {
            steps {
                script {
                    env.GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                }
            }
        }
        
        stage('Build & JUnit Test') {
            steps {
                sh 'mvn clean install'
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT} .'
            }
        }
        
        stage('Docker Image Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    sh 'docker push ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}'
                    sh 'docker rmi ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}'
                }
            }
        }
        
        stage('Clone/Pull k8s deployment Repo') {
            steps {
                script {
                    if (fileExists('DevOps_Masterpiece-Project')) {
                        echo 'Cloned repo already exists - Pulling latest changes'
                        dir("DevOps_Masterpiece-Project") {
                            sh 'git config pull.rebase true'
                            sh 'git pull --rebase'
                        }
                    } else {
                        echo 'Repo does not exist - Cloning the repo'
                        sh 'git clone -b feature https://github.com/MAJJISAGAR/DevOps_Masterpiece-Project.git'
                    }
                }
            }
        }
        
        stage('Update deployment Manifest') {
            steps {
                dir("DevOps_Masterpiece-Project/yamls") {
                    sh 'sed -i "s#sagar520.*#${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}#g" deployment.yaml'
                    sh 'cat deployment.yaml'
                }
            }
        }
        
        stage('Commit & Push changes to feature branch') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    dir("DevOps_Masterpiece-Project/yamls") {
                        sh "git config --global user.email 'sagarmajji143@gmail.com'"
                        sh "git config --global user.name 'MAJJISAGAR'"
                        sh 'git remote set-url origin https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}'
                        sh 'git checkout feature'
                        sh 'git add deployment.yaml'
                        sh "git commit -am 'Updated image version for Build- ${VERSION}-${GIT_COMMIT}'"
                        sh 'git push origin feature'
                    }
                }
            }
        }
        
        stage('Raise PR') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    dir("DevOps_Masterpiece-Project/yamls") {
                        sh '''
                            unset GITHUB_TOKEN
                            echo "${GITHUB_TOKEN}" | gh auth login --with-token
                        '''
                        sh 'git checkout feature'
                        sh "gh pr create -t 'Image tag updated' -b 'Check and merge it'"
                    }
                }
            }
        }
    }
}
