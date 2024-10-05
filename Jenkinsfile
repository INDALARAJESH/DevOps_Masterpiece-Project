pipeline {
    agent any

    environment {
        NAME = "spring-app"
        VERSION = "${env.BUILD_ID}"
        IMAGE_REPO = "indalarajesh"
        GIT_REPO_NAME = "e2e-project"
        GIT_USER_NAME = "INDALARAJESH"
    }

    tools { 
        maven 'maven-3.8.6' 
    }

    stages {
        stage('Checkout git') {
            steps {
                git branch: 'main', url: 'https://github.com/INDALARAJESH/e2e-project.git'
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
                    if (fileExists('e2e-project')) {
                        echo 'Cloned repo already exists - Pulling latest changes'
                        dir("e2e-project") {
                            sh 'git pull'
                        }
                    } else {
                        echo 'Repo does not exist - Cloning the repo'
                        sh 'git clone -b feature https://github.com/INDALARAJESH/e2e-project.git'
                    }
                }
            }
        }
        
        stage('Update deployment Manifest') {
            steps {
                dir("e2e-project/yamls") {
                    sh 'sed -i "s#indalarajesh.*#${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}#g" deployment.yaml'
                    sh 'cat deployment.yaml'
                }
            }
        }
        
        stage('Commit & Push changes to feature branch') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    dir("e2e-project/yamls") {
                        sh "git config --global user.email 'rajeshindala1997@gmail.com'"
                        sh "git config --global user.name 'INDALARAJESH'"
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
                    dir("e2e-project/yamls") {
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
