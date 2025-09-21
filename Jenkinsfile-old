pipeline {
    agent any

    environment {
        APP_NAME = "nodeapp"
        MAIN_PORT = "3000"
        DEV_PORT  = "3001"
    }

    tools {
        nodejs 'node7.8.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "Tests failed but continuing..."'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker build -t nodemain:v1.0 ."
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh "docker build -t nodedev:v1.0 ."
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh '''
                        docker ps -q --filter "ancestor=nodemain:v1.0" | xargs -r docker stop
                        docker ps -a -q --filter "ancestor=nodemain:v1.0" | xargs -r docker rm
                        docker run -d --name nodemain -p 3000:3000 nodemain:v1.0
                        '''
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh '''
                        docker ps -q --filter "ancestor=nodedev:v1.0" | xargs -r docker stop
                        docker ps -a -q --filter "ancestor=nodedev:v1.0" | xargs -r docker rm
                        docker run -d --name nodedev -p 3001:3000 nodedev:v1.0
                        '''
                    }
                }
            }
        }
    }
}
